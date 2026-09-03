# 02. Cache Stampede (Dogpiling) & Probabilistic Early Expiration

## 1. Problem
A hot cache key (e.g. homepage banner serving 50,000 QPS) expires. Within 10 milliseconds, 5,000 concurrent threads suffer a cache miss simultaneously and all hammer the database with the exact same expensive SQL query (**Cache Stampede / Dogpiling**), crashing the database cluster.

## 2. Production Context
Cache Stampedes occur when a heavily-read cached item expires while under high concurrency.

## 3. Mental Model: The XFetch Probabilistic Early Expiration Algorithm
Instead of waiting for the key to expire at $t = \text{TTL}$, a reader asynchronously recomputes and refreshes the key *before* expiration with a probability that increases as the key approaches its TTL.

$$\mathbf{Recompute\ Condition} \iff -\beta \times \delta \times \ln(\mathbf{random}()) > \mathbf{TTL} - \text{Current Time}$$
- $\beta > 0$: Aggressiveness parameter (default: $1.0$).
- $\delta$: Execution time required to compute the database query (in seconds).
- $\text{random}()$: Uniform random number $\in (0, 1]$.

```mermaid
flowchart TD
    READ[Read Request for Key] --> XFETCH{Evaluate XFetch Probabilistic Function}
    XFETCH -->|True: Probability Threshold Met (Key near expiry)| MUTEX{Acquire Distributed Lock}
    XFETCH -->|False: Key Still Fresh| RETURN_CACHE[Return Cached Value in 1ms]
    
    MUTEX -->|Lock Acquired| ASYNC_RECOMPUTE[Background Thread Recomputes DB & Updates Redis]
    MUTEX -->|Lock Contended| RETURN_CACHE
    ASYNC_RECOMPUTE --> RETURN_CACHE
```

---

## 4. Distributed Mutex Lock Defense (Single-Flight Pattern)

If probabilistic refresh is not used, enforce **Single-Flight Lock Synchronization** on cache misses:

```java
public String getProduct(String key) {
    String value = redis.get(key);
    if (value != null) return value;

    // Cache Miss: Acquire short 5-second distributed mutex lock in Redis
    String lockKey = "lock:" + key;
    if (redis.set(lockKey, "1", "NX", "EX", 5)) {
        try {
            // Only 1 thread hits the DB to recompute!
            value = database.fetchProduct(key);
            redis.setex(key, 3600, value);
            return value;
        } finally {
            redis.del(lockKey);
        }
    } else {
        // Other 4,999 threads sleep 50ms and retry cache lookup!
        Thread.sleep(50);
        return getProduct(key);
    }
}
```

---

## 5. Interview Questions & Model Answers

**Q1: What is a Cache Stampede (Dogpiling), and how do you mitigate it in a system receiving 100,000 QPS?**
**Answer**: A Cache Stampede occurs when a hot cache key expires or is evicted while under high concurrency, causing thousands of concurrent worker threads to miss the cache simultaneously and all execute the same expensive database query at once, overwhelming database CPU and connection pools. To mitigate this at 100k QPS:
1. **Single-Flight / Distributed Mutex Locking**: Use `SET lock:key NX EX 5` so that only the first thread executes the database query while all other concurrent requests wait and read the repopulated cache.
2. **Probabilistic Early Expiration (XFetch algorithm)**: Recompute and refresh the cache key asynchronously in the background before its hard TTL expires based on query execution time $\delta$ and elapsed time.
3. **Never-Expiring Background Cache with Async Refresh**: Set key without TTL in Redis and use a background scheduled worker (or Kafka consumer) to update it periodically.

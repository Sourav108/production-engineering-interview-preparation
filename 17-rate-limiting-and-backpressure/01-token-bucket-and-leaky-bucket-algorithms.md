# 01. Token Bucket, Leaky Bucket, and Distributed Rate Limiting

## 1. Problem
A surge of traffic from a rogue client, scraping bot, or flash sale overwhelms backend services, driving CPU to 100% and taking down the API for all legitimate users.

## 2. Production Context
Rate limiting protects system capacity by enforcing hard traffic bounds per IP, API key, user ID, or global service route.

## 3. Mental Model: The Core Rate Limiting Algorithms

| Algorithm | How It Works | Burst Behavior | Ideal Use Case |
| :--- | :--- | :--- | :--- |
| **Token Bucket** | Tokens added at steady rate $R$; requests consume tokens. Bucket capacity $B$. | **Allows bursts up to bucket capacity $B$** | General API rate limiting (allows short traffic spikes) |
| **Leaky Bucket** | Requests enter queue and leak out at a strictly constant rate. | **Smooths bursts to a constant output stream** | Downstream packet pacing, preventing bursts |
| **Sliding Window Counter** | Blends previous window count with current window percentage. | Bounded burst at window boundary | Memory-efficient distributed rate limiting |

---

## 4. Atomic Redis Token Bucket via Lua Script

To prevent race conditions across distributed API Gateway instances, evaluate token refill atomically in Redis:

```lua
-- KEYS[1]: Rate limit key (e.g. rate_limit:user_123)
-- ARGV[1]: Max bucket capacity (e.g. 100)
-- ARGV[2]: Refill rate per second (e.g. 10)
-- ARGV[3]: Current timestamp (epoch seconds)
-- ARGV[4]: Tokens requested (e.g. 1)

local key = KEYS[1]
local capacity = tonumber(ARGV[1])
local refill_rate = tonumber(ARGV[2])
local now = tonumber(ARGV[3])
local requested = tonumber(ARGV[4])

local data = redis.call('HMGET', key, 'tokens', 'last_updated')
local tokens = tonumber(data[1])
local last_updated = tonumber(data[2])

if tokens == nil then
    tokens = capacity
    last_updated = now
else
    -- Refill tokens based on elapsed time
    local elapsed = math.max(0, now - last_updated)
    tokens = math.min(capacity, tokens + elapsed * refill_rate)
    last_updated = now
end

if tokens >= requested then
    tokens = tokens - requested
    redis.call('HMSET', key, 'tokens', tokens, 'last_updated', last_updated)
    redis.call('EXPIRE', key, math.ceil(capacity / refill_rate))
    return 1 -- ALLOWED
else
    return 0 -- REJECTED (HTTP 429 Too Many Requests)
end
```

---

## 5. Standard HTTP 429 Response Headers
When rate limiting a client, return structured standard headers:
- `HTTP/1.1 429 Too Many Requests`
- `Retry-After: 30` (Seconds client must wait)
- `X-RateLimit-Limit: 100` (Maximum requests allowed)
- `X-RateLimit-Remaining: 0` (Remaining quota in current window)
- `X-RateLimit-Reset: 1725372000` (Unix epoch timestamp when bucket refills)

---

## 6. Interview Questions & Model Answers

**Q1: What is the difference between Token Bucket and Leaky Bucket?**
**Answer**: A **Token Bucket** algorithm accumulates tokens at a constant rate up to a maximum bucket capacity: when a traffic burst arrives, it can consume all available tokens instantly and proceed without delay, making it ideal for user-facing APIs that naturally experience short bursty patterns. A **Leaky Bucket** acts as a FIFO queue with a fixed leak rate: incoming requests queue up and are dispatched to the backend at a strictly constant rate regardless of burst size. Any requests arriving when the queue is full are dropped, making it ideal for pacing outbound traffic to fragile downstream dependencies that cannot tolerate bursts.

**Q2: Why must distributed rate limiters be executed via Redis Lua scripts rather than separate GET and SET commands?**
**Answer**: In a distributed environment with multiple API Gateway instances, executing `GET key` followed by application-side math and `SET key` introduces a classic **Check-Then-Act Race Condition**. If 50 concurrent requests hit 10 gateway instances simultaneously, all 10 gateways will read the same token count and approve the requests, permitting $50\times$ more traffic than the configured rate limit. A Redis Lua script executes **atomically on the Redis single-threaded event loop**, guaranteeing that reading, refilling, decrementing, and updating timestamps occur with zero race conditions.

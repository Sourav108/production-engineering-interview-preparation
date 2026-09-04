# 04. Production System Design: Large-Scale Distributed Search

## 1. System Requirements & Functional Scope
- **Functional**: Full-text indexing, filtering, faceted search, and ranking across 500 million product documents with sub-second document indexing.
- **Availability Target**: $99.95\%$ Availability.
- **Latency SLO**: $p99 \le 100\text{ms}$ under $50,000\text{ QPS}$ search traffic.

---

## 2. Architecture: Fan-Out & Scatter-Gather Pipeline

```mermaid
flowchart TD
    CLIENT[User Search Query: 'wireless noise cancelling headphones'] --> EDGE[Edge Ingress Gateway]
    EDGE --> CACHE{Redis L1 Cache: Popular Queries}
    CACHE -->|Hit: 80% Traffic| FAST_RESP[Return Cached Search Results in 2ms]
    
    CACHE -->|Miss: 20% Traffic| AGG[Search Aggregator Node]
    
    subgraph 50-Shard Distributed Search Cluster (Elasticsearch / Lucene)
        AGG -->|Parallel Fan-Out| SHARD_1[Shard 1 Index Search]
        AGG -->|Parallel Fan-Out| SHARD_2[Shard 2 Index Search]
        AGG -->|Parallel Fan-Out| SHARD_N[Shard 50 Index Search]
    end
    
    SHARD_1 --> MERGE[Aggregator: Top-K Score Merge & Ranking]
    SHARD_2 --> MERGE
    SHARD_N --> MERGE
    
    MERGE --> ASYNC_CACHE[Asynchronously Populate Redis Cache]
    MERGE --> USER_RESP[Return Top 20 Ranked Products]
```

---

## 3. Mitigating Fan-Out Tail Latency Amplification
- **Hedged Requests**: When the Aggregator dispatches queries to 50 shards in parallel, if any single shard fails to return results within the expected **p95 threshold (25ms)**, a duplicate hedged query is dispatched to that shard's secondary replica.
- **Partial Availability / Soft Degradation**: If 1 of the 50 shards times out after a hard 80ms deadline, the Aggregator merges results from the 49 responsive shards and returns the best results with a response header:
  `X-Search-Status: DEGRADED_PARTIAL_SHARD_TIMEOUT`
  *(Users see 98% of relevant products in 80ms rather than a 504 error!).*

---

## 4. Cache Stampede Defense via XFetch
- Popular search queries (e.g. "iPhone", "Nike Shoes") are cached in Redis and asynchronously refreshed using the **XFetch Probabilistic Early Expiration Algorithm** before their 10-minute TTL expires, ensuring the 50-shard Elasticsearch cluster is never exposed to 100% uncached query volume.

---

## 5. Trade-offs & Production Defense
- **Partial Result Completeness vs Latency**: Returning 98% complete search results in 80ms provides a vastly superior user experience compared to stalling the user for 5,000ms waiting for a single slow shard to complete garbage collection.

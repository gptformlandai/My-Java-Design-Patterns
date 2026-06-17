# Caching Pattern

Category: Reliability, Resilience, and Operations Patterns  
MAANG interview meter: Very High  
Software usage meter: Very High  
Repository module: [github-repo/caching](../../github-repo/caching)

## How to Study This Page

Use this page in three passes:

1. First pass: understand cache as a faster copy of data or computation.
2. Second pass: trace cache hit, cache miss, fill, eviction, and invalidation.
3. Third pass: compare cache-aside, read-through, write-through, write-behind, and write-around.

By the end, you should be able to say:

> Caching stores frequently used data in faster storage to reduce latency and load on the source system.

## 1. Technical Definition

Caching is a performance and resilience pattern where frequently accessed or expensive-to-compute data is stored in a faster layer for reuse.

Core idea:

- Cache hit returns data quickly.
- Cache miss falls back to source of truth.
- Cache fill stores data for future requests.
- Eviction removes old or less useful entries.
- Invalidation keeps cache from serving stale data too long.

### 30-Second Interview Answer

I would use caching when reads are frequent, data is expensive to compute or fetch, and slightly stale data is acceptable. I would define the cache key, TTL, eviction policy, invalidation strategy, and fallback to source of truth. The trade-off is freshness versus latency: caching improves performance and reduces load, but stale data, cache stampede, and consistency bugs must be managed.

## 2. Layman and Easy to Understand Definition

Caching is like keeping frequently used notes on your desk instead of walking to the archive every time.

The desk copy is faster. But if the archive changes, you need a way to refresh or discard the desk copy.

In code:

- Look in cache first.
- If found, return it.
- If not found, fetch from source.
- Store it in cache.
- Return it.

## 3. Bit by Bit Explanation of What It Is

### 3.1 The Problem

Repeatedly calling slow resources creates high latency and load:

```text
request -> database -> same result
request -> database -> same result
request -> database -> same result
```

If many users ask for the same data, the source system does unnecessary work.

### 3.2 The Caching Solution

Add a faster layer:

```text
request -> cache hit -> response
request -> cache miss -> database -> cache fill -> response
```

### 3.3 Main Participants

| Participant | Meaning |
|---|---|
| Cache key | Unique identifier for cached data. |
| Cache value | Stored result or object. |
| Source of truth | Database, service, or computation that owns correct data. |
| TTL | Time after which cached entry expires. |
| Eviction policy | Rule for removing entries, such as LRU. |
| Invalidation | Explicit removal or update of stale data. |

### 3.4 Common Strategies

| Strategy | How it works |
|---|---|
| Cache-aside | Application checks cache, then loads source on miss. |
| Read-through | Cache layer loads source on miss. |
| Write-through | Write updates cache and source together. |
| Write-around | Write source directly and load cache later. |
| Write-behind | Write cache first, asynchronously write source later. |

## 4. Java Coding Example

This example shows cache-aside with a simple TTL.

```java
import java.time.Instant;
import java.util.HashMap;
import java.util.Map;
import java.util.function.Function;

record CacheEntry(String value, Instant expiresAt) {
}

class TtlCache {
    private final Map<String, CacheEntry> cache = new HashMap<>();

    String get(String key, Function<String, String> loader) {
        CacheEntry entry = cache.get(key);
        if (entry != null && entry.expiresAt().isAfter(Instant.now())) {
            return entry.value();
        }

        String loaded = loader.apply(key);
        cache.put(key, new CacheEntry(loaded, Instant.now().plusSeconds(60)));
        return loaded;
    }

    void invalidate(String key) {
        cache.remove(key);
    }
}
```

### Java Block by Block

| Code part | What it teaches |
|---|---|
| `CacheEntry` | Cache stores value plus expiration. |
| `get` | Implements cache-aside read. |
| `entry.expiresAt` | Prevents serving very old data. |
| `loader` | Source of truth fallback. |
| `invalidate` | Explicit stale data removal. |

### Java Usage

```java
TtlCache cache = new TtlCache();

String user = cache.get("user:42", key -> "loaded-from-database");
System.out.println(user);
```

## 5. Python Coding Example

```python
import time


class TtlCache:
    def __init__(self, ttl_seconds):
        self.ttl_seconds = ttl_seconds
        self.store = {}

    def get(self, key, loader):
        entry = self.store.get(key)
        now = time.time()
        if entry and entry["expires_at"] > now:
            return entry["value"]

        value = loader(key)
        self.store[key] = {"value": value, "expires_at": now + self.ttl_seconds}
        return value


cache = TtlCache(ttl_seconds=60)
print(cache.get("user:42", lambda key: "loaded-from-database"))
```

### Python Usage

Use this shape when explaining:

- Cache miss calls the source of truth.
- TTL controls freshness.
- Key design determines correctness.

## 6. Where It Comes Handy in Real Life

- User profile lookups.
- Product catalog pages.
- Session data.
- API responses.
- Computed recommendations.
- Permission checks.
- CDN static content.
- Database query results.

## 7. Advantages Over Normal Code Without Pattern

Without caching:

```text
every request hits slow source
```

With caching:

```text
hot requests are served from fast storage
```

Benefits:

- Lower latency.
- Higher throughput.
- Lower database load.
- Better resilience during brief source slowness.
- Lower compute cost.

## 8. Where It Excels

- Read-heavy workloads.
- Data changes less often than it is read.
- Some staleness is acceptable.
- Source system is expensive or slow.
- Hot keys or repeated computations exist.

## 9. Where It Fails

- Data must be perfectly fresh.
- Cache key is poorly designed.
- Invalidations are forgotten.
- Hot key stampede overloads source on expiry.
- Cache memory is undersized.
- Cached sensitive data lacks access control.

For strict correctness, prefer source-of-truth reads or carefully designed write-through/invalidation.

## 10. Prebuilt Frameworks and Packages

| Ecosystem | Options |
|---|---|
| Java local cache | Caffeine, Guava Cache, Ehcache |
| Distributed cache | Redis, Memcached, Hazelcast, Apache Ignite |
| Spring | Spring Cache abstraction |
| CDN | CloudFront, Fastly, Akamai, Cloudflare |
| Patterns | Cache-aside, read-through, write-through, write-behind |

## 11. Pros and Cons

| Pros | Cons |
|---|---|
| Reduces latency. | Can serve stale data. |
| Lowers backend load. | Adds invalidation complexity. |
| Improves throughput. | Can cause cache stampede. |
| Reduces repeated computation. | Uses extra memory and infrastructure. |

## 12. Real-World Identification Example

Scenario: Product detail API reads the same popular product thousands of times per minute.

Caching fit:

- Cache key is `product:{id}`.
- TTL is short enough for product updates.
- Cache-aside loads from database on miss.
- Invalidation happens when product changes.

Without it:

- Database handles repeated identical reads.
- Product page latency rises during traffic spikes.

## 13. MAANG Interview Triggers

Use Caching when you hear:

- "How do we reduce read latency?"
- "Database is overloaded."
- "Same data is requested repeatedly."
- "How do we handle hot keys?"
- "Can stale data be tolerated?"
- "How do we scale reads?"

Strong answer keywords:

- cache hit
- cache miss
- TTL
- eviction
- invalidation
- cache-aside
- write-through
- cache stampede
- hot key

## 14. Common Mistakes

### Mistake 1: No invalidation strategy

- Why it is wrong: users see stale or wrong data.
- Better approach: define TTL plus explicit invalidation on writes.

### Mistake 2: Bad cache key design

- Why it is wrong: different users or permissions can share incorrect data.
- Better approach: include tenant, user, locale, and authorization dimensions when needed.

### Mistake 3: Ignoring cache stampede

- Why it is wrong: many requests miss at once and overload source.
- Better approach: use request coalescing, jittered TTLs, locks, or stale-while-revalidate.

### Mistake 4: Caching errors forever

- Why it is wrong: transient failures become persistent responses.
- Better approach: cache failures only briefly, if at all.

## 15. Caching vs Similar Patterns

| Pattern | Difference |
|---|---|
| Caching | Stores reusable data for faster future access. |
| Replication | Copies source data for availability and read scaling. |
| CDN | Edge caching for static or HTTP content. |
| Materialized View | Precomputed read model stored as durable data. |
| Memoization | In-process caching of function results. |

## 16. Caching Design Checklist

- What is the cache key?
- What is the source of truth?
- What is acceptable staleness?
- What TTL is used?
- What eviction policy is used?
- How are writes handled?
- How is invalidation triggered?
- How are hot keys protected?
- What metrics show hit rate and miss latency?

## 17. Quick Revision Notes

- One-line summary: Caching trades freshness complexity for lower latency and lower load.
- Three keywords: hit, TTL, invalidation.
- Interview trap: saying "add Redis" without explaining consistency and stampede handling.
- Memory trick: fast copy in front of slow truth.

## 18. Mini Exercise

Design caching for a product catalog.

Answer these:

1. What is the cache key?
2. What TTL is safe?
3. How are product updates invalidated?
4. How do you protect hot products?
5. What metric proves the cache helps?

## 19. Source Reference in This Repo

Study these files after reading the concept:

- [github-repo/caching/README.md](../../github-repo/caching/README.md)
- [github-repo/caching/src/main/java/com/iluwatar/caching/CacheStore.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/CacheStore.java)
- [github-repo/caching/src/main/java/com/iluwatar/caching/LruCache.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/LruCache.java)
- [github-repo/caching/src/main/java/com/iluwatar/caching/CachingPolicy.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/CachingPolicy.java)
- [github-repo/caching/src/main/java/com/iluwatar/caching/UserAccount.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/UserAccount.java)
- [github-repo/caching/src/main/java/com/iluwatar/caching/database/DbManager.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/database/DbManager.java)
- [github-repo/caching/src/main/java/com/iluwatar/caching/App.java](../../github-repo/caching/src/main/java/com/iluwatar/caching/App.java)

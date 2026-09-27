# Day 04 — Caching

## 1. What is Caching?

Caching means storing frequently accessed data in a faster storage layer so future requests can be served faster.

Without caching:

User → Application → Database

With caching:

User → Application → Cache → Database

If the data is already in the cache:

User → Application → Cache → Response

The database doesn't need to be accessed.

---

## 2. Why Do We Need Caching?

Suppose an application receives 10,000 requests/sec and many requests ask for the same data.

Without caching:

10,000 requests
       ↓
   Database

With caching:

10,000 requests
       ↓
      Cache
       ↓
Most requests served here

Only cache misses
       ↓
    Database

Caching can:

- reduce database load
- reduce latency
- improve throughput
- reduce repeated database queries
- reduce expensive computation

---

## 3. Cache Hit and Cache Miss

### Cache Hit

The requested data exists in the cache.

Request → Cache → Data → Response

### Cache Miss

The requested data does not exist in the cache.

Request → Cache → MISS
                  ↓
              Database
                  ↓
             Store in Cache
                  ↓
               Response

---

## 4. Cache Hit Rate

Cache hit rate tells us how often requests are successfully served from the cache.

Cache Hit Rate = Cache Hits / Total Cache Requests

Example:

1000 cache requests
900 hits

Hit rate = 90%

A high hit rate generally means the cache is handling a large portion of requests.

But hit rate isn't the only metric.

We also care about:

- latency
- database load
- memory usage
- freshness
- eviction
- failure behavior

---

## 5. Where Can Caching Exist?

Caching can happen at multiple layers:

User
 ↓
Browser Cache
 ↓
CDN
 ↓
Reverse Proxy
 ↓
Application Cache / Redis
 ↓
Database

Different layers solve different problems.

Examples:

- Browser cache → avoid downloading the same resource again
- CDN → serve content closer to users
- Redis → shared application-level cache
- Database buffer/cache → reduce expensive disk access

---

## 6. Redis as an Application Cache

Redis is commonly used as a fast, shared application-level cache.

Example:

Application
     ↓
   Redis
     ↓
PostgreSQL

Example key:

user:123

Value:

{
    "id": 123,
    "name": "Ayush"
}

Redis is useful because it provides very fast access to frequently used data.

---

## 7. Cache-Aside Pattern

A common caching pattern is Cache-Aside.

Flow:

Application
     ↓
Check Cache
     ↓
   Hit? ── Yes → Return data
     │
     No
     ↓
Database
     ↓
Store result in Cache
     ↓
Return data

The application decides when to read from and populate the cache.

This is one of the most common application caching patterns.

---

## 8. Cache Invalidation

Suppose PostgreSQL contains:

user:123
name = Ayush

Redis also contains:

user:123
name = Ayush

Now the user changes their name:

Ayush → Rahul

PostgreSQL is updated:

name = Rahul

But Redis may still contain:

name = Ayush

The cache is now stale.

This creates the cache invalidation problem.

Possible approaches:

- delete the cache entry after updating the database
- update the cache after updating the database
- use TTL
- event-driven invalidation
- versioning

---

## 9. TTL — Time To Live

A cache entry can automatically expire after a specified time.

Example:

user:123
TTL = 300 seconds

After 5 minutes, the entry expires.

Next request:

Cache Miss
    ↓
Database
    ↓
Fresh Data
    ↓
Cache again

TTL is useful when some amount of stale data is acceptable.

---

## 10. TTL Trade-off

Short TTL:

Freshness ↑
Cache hit rate ↓
Database load ↑

Long TTL:

Cache hit rate ↑
Database load ↓
Risk of stale data ↑

Therefore, there is no universally correct TTL.

The correct value depends on the data and system requirements.

---

## 11. Cache Eviction

A cache has finite memory.

When it becomes full, data may need to be removed.

Common strategies include:

### LRU — Least Recently Used

Remove data that has not been accessed recently.

### LFU — Least Frequently Used

Remove data that is accessed less frequently.

### TTL-based expiration

Remove data after its expiration time.

The correct strategy depends on the workload.

---

## 12. Cache vs Source of Truth

A useful default architecture is:

Source of Truth
      ↓
PostgreSQL
      ↓
Cache
      ↓
Fast Reads

The cache should generally not be the only copy of important business data.

If the cache disappears:

Redis ❌

the application should ideally be able to reconstruct the cached data from the source of truth.

---

## 13. Cache Consistency

Caching creates another important question:

> What happens when the cache and database contain different values?

Example:

PostgreSQL:
balance = ₹1000

Redis:
balance = ₹800

Which value should the application trust?

For critical information such as:

- payments
- account balances
- financial transactions

we must carefully design caching and consistency.

The architecture should define:

- source of truth
- acceptable staleness
- invalidation strategy
- failure behavior

---

## 14. Cache Failure

Suppose:

Application
    ↓
Redis ❌
    ↓
PostgreSQL

If Redis is only a cache, the application may fall back to PostgreSQL.

But now many requests may suddenly hit the database:

All Requests
     ↓
PostgreSQL

This can dramatically increase database load.

Therefore cache failure must be considered when designing the system.

---

## 15. Cache Stampede

Suppose a popular cache entry expires:

user:123 → expired

Thousands of requests arrive at almost the same time.

They all see:

CACHE MISS

and all query the database.

1000 requests
      ↓
PostgreSQL
      ↓
Potential overload

This is called a cache stampede.

Possible solutions include:

- request coalescing
- locking
- staggered expiration
- background refresh
- stale-while-revalidate

The goal is to prevent many requests from rebuilding the same cache entry simultaneously.

---

## 16. Caching Is Not Always Good

Caching adds complexity.

Potential problems:

- stale data
- cache invalidation complexity
- memory limits
- cache misses
- cache stampedes
- additional infrastructure
- consistency problems

Therefore:

> Don't add a cache simply because caching makes systems faster.

Ask:

1. Is the data read frequently?
2. Is retrieving the data expensive?
3. Can some staleness be tolerated?
4. What happens if the cache fails?
5. What is the source of truth?
6. How will the cache be invalidated?

---

# Key Takeaways

1. Caching stores frequently accessed data in a faster layer.
2. Cache hits avoid expensive database or computation work.
3. Cache misses require retrieving the data from the source.
4. Cache hit rate measures how often requests are served from cache.
5. Redis is commonly used for application-level caching.
6. Cache-Aside is a common caching pattern.
7. Cache invalidation is required when cached data becomes stale.
8. TTL automatically expires cached entries.
9. Short TTL improves freshness but can increase database load.
10. Long TTL improves hit rate but can increase staleness.
11. Caches have finite memory and require eviction strategies.
12. PostgreSQL can act as the durable source of truth for business data.
13. Cache failure can suddenly increase database traffic.
14. Cache stampedes can overload the database when many requests rebuild the same cache entry.
15. Caching is a trade-off between latency, database load, freshness, memory, and complexity.

---

# Mental Model

Without caching:

Users
  ↓
Application
  ↓
Database

With caching:

Users
  ↓
Application
  ↓
Cache
  ↓
Database

Cache hit:

Users
  ↓
Application
  ↓
Cache
  ↓
Response

Cache miss:

Users
  ↓
Application
  ↓
Cache → MISS
  ↓
Database
  ↓
Cache
  ↓
Response

Core idea:

> Caching trades memory and consistency complexity for lower latency and reduced load on slower systems.
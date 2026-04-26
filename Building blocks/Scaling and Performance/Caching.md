Caching for Senior and Staff-Level Interviews

Caching is one of the most common answers in system design because it directly improves latency, reduces load on downstream systems, and helps systems scale. At senior and staff level, the important skill is not just saying "add Redis." It is knowing what to cache, where to cache it, how to invalidate it, and what correctness risks caching introduces.

1. What caching is

A cache stores data closer to the client or application so repeated reads can be served faster than going back to the original system of record every time.

Common cache locations:

- browser cache
- CDN or edge cache
- reverse proxy cache
- application in-memory cache
- distributed cache such as Redis or Memcached
- database internal cache or buffer cache

The key idea:

- expensive or repeated reads are served from a faster layer

2. Why caching exists

Caching helps with:

- lower latency
- lower database load
- better throughput
- lower infrastructure cost
- resilience during traffic spikes

Real examples:

- caching product pages
- caching user profile lookups
- caching feature flags
- caching expensive analytics queries
- caching authentication/session state

3. When to use caching

Caching is a strong fit when:

- the same data is read repeatedly
- reads greatly outnumber writes
- slight staleness is acceptable
- the original source is expensive or slow

Caching is dangerous when:

- correctness depends on seeing the very latest value
- the invalidation strategy is unclear
- the workload has poor locality and low cache hit rate

4. Important cache metrics

Cache hit:

- the requested data is already in cache, so the system serves it directly

Cache miss:

- the data is not in cache, so the system must fetch it from the source of truth

Hit rate:

- the percentage of requests served from cache
- a high hit rate usually means the cache is useful

Eviction:

- removing data from cache to make room for other data

TTL:

- time to live
- how long a cached value stays valid before expiring

Why these matter:

- a cache with a poor hit rate may not be worth its complexity

5. Common caching patterns

Cache-aside:

- the application checks the cache first
- if the value is missing, it reads from the database and then stores the result in cache

Why it is common:

- simple
- flexible
- the application controls what gets cached

Downside:

- the first miss is slower
- stale values remain until TTL expiry or invalidation

Write-through:

- writes go to the cache and the backing store together

Benefit:

- cache stays warm and consistent after writes

Downside:

- more write latency
- wasted work for rarely read data

Write-behind or write-back:

- write goes to the cache first and is flushed to the backing store later

Benefit:

- fast writes

Downside:

- higher durability risk if the cache fails before persistence happens

Read-through:

- the cache layer itself fetches missing values from the backing store

Benefit:

- simpler application code in some architectures

6. Cache invalidation

Cache invalidation is one of the hardest parts of caching.

Why:

- if the cache is not updated or cleared when the source changes, clients may see stale data

Common strategies:

- TTL-based expiration
- explicit deletion on write
- versioned keys
- event-driven invalidation

TTL-based expiration:

- easiest to operate
- acceptable when slight staleness is fine

Explicit invalidation:

- delete or update cached keys when the underlying data changes
- better freshness, but more complexity

Versioned keys:

- include a version number or timestamp in the cache key
- useful when old cached values can safely expire on their own

7. What to cache

Good candidates:

- product catalogs
- profile lookups
- configuration and feature flags
- read-heavy query results
- session data
- rendered pages or fragments

Bad candidates:

- rapidly changing balances
- highly personalized data with little reuse
- data with low repeat access
- write-heavy objects that are expensive to keep consistent

8. Where to cache

Browser cache:

- best for static assets and client-reusable responses

CDN or edge cache:

- best for globally distributed content and public cacheable responses

Application memory cache:

- best for very fast per-instance lookups
- downside is per-instance inconsistency and limited memory

Distributed cache:

- best when many app servers need the same cached values
- Redis is the most common example

Database cache:

- often automatic, but still not a substitute for good application-level cache design

9. Common technologies

Redis:

- the most common distributed cache answer in interviews
- supports strings, hashes, sets, sorted sets, TTLs, and pub/sub
- used for session storage, rate limiting, counters, leaderboards, and cache data

Memcached:

- simple in-memory distributed cache
- best for straightforward key-value caching without Redis-style extra features

CDNs:

- Cloudflare, Fastly, Akamai, CloudFront
- best for caching static and edge-deliverable content

10. Common cache problems

Cache stampede:

- many requests miss the same key at once and all hit the database

Ways to reduce it:

- request coalescing
- locking around refill
- jittered TTLs

Hot key:

- one key becomes extremely popular and overloads one cache node or shard

Ways to reduce it:

- replicate hot keys
- split traffic
- precompute or special-case hot content

Cache penetration:

- repeated requests for missing data keep hitting the source

Ways to reduce it:

- negative caching
- bloom filters in some systems

11. Consistency tradeoffs

Caches improve performance by accepting some staleness risk.

Important question:

- how stale can the data be before it becomes harmful?

Examples:

- product description can usually be stale for a little while
- inventory count often cannot

Senior/staff-level point:

- caching is a business correctness decision, not just a latency optimization

12. Real examples

Use caching for:

- product and content lookups
- session and token metadata
- expensive search results
- dashboards with tolerable staleness
- API response acceleration

Be careful caching:

- money movements
- inventory reservation counts
- permission checks immediately after updates

13. Staff-level discussion prompts

- What exactly are we caching and why?
- Where should the cache live?
- How stale can the data be?
- What is the invalidation strategy?
- What happens during cache stampede or cache node failure?
- Is the hit rate high enough to justify the complexity?

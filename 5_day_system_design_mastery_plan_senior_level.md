# 5-Day System Design Mastery Plan (Senior / Staff Level)

---

## 🎯 Objective

This plan is designed for engineers who already understand system design fundamentals and want to **operate at a senior/staff level in interviews**.

Focus is NOT on learning concepts, but on:
- Making strong architectural decisions
- Articulating tradeoffs clearly
- Handling edge cases and failures
- Demonstrating real-world engineering judgment

---

## 🧠 Core Interview Framework (Memorize This)

Every answer should follow this structure:

1. Requirements
   - Functional (features)
   - Non-functional (scale, latency, consistency)

2. High-Level Design
   - Major components
   - Data flow

3. Data Model
   - Schema or access patterns

4. API Design
   - Endpoints, contracts

5. Deep Dives
   - Scaling
   - Tradeoffs
   - Failure handling

---

## 🔥 DAY 1 — API Design + Data Modeling

### 🎯 Goal
Become elite at translating requirements into clean APIs and schemas.

---

### 1. API Design (High Signal Area)

#### Key Principles

- Idempotency
  - Required for POST operations that may retry
  - Prevent duplicate side effects

- Pagination
  - Prefer cursor-based pagination over offset
  - Offset breaks at scale and with mutations

- Versioning
  - Use `/v1/` or header-based versioning

- Filtering & Sorting
  - Keep flexible but indexed

- Consistency of contracts
  - Predictable request/response shapes

---

### Example: Todo API

```
POST /todos
GET /todos?cursor=abc&limit=20
PATCH /todos/{id}/position
DELETE /todos/{id}
```

#### Senior-Level Talking Points

- Cursor pagination avoids skipping/duplicate items
- PATCH used for partial updates (position only)
- Idempotency key for create requests

---

### 2. Data Modeling

#### Relational (Postgres)

Use when:
- Strong consistency required
- Transactions matter
- Relationships exist

Example:

```
Todos
- id (PK)
- user_id
- title
- position
- created_at
```

---

#### NoSQL (DynamoDB / Cassandra)

Use when:
- Massive scale
- Predictable access patterns

Design rule:
👉 Model based on queries, NOT normalization

Example (Feed Table):

```
PK: user_id
SK: timestamp
```

---

### Tradeoffs Table

| DB        | Strength       | Weakness        |
|----       |--------        |--------        |
| Postgres  | ACID, joins    | scaling writes |
| DynamoDB  | scale, latency | rigid queries |
| Cassandra | write-heavy    | weak consistency |
| Redis     | fast           | memory-bound |

---

### Practice Exercises

- Design schema for:
  - Chat system
  - Notification system
  - Rate limiter

---

## ⚙️ DAY 2 — Concurrency + Race Conditions

### 🎯 Goal
Handle real-world edge cases confidently.

---

### 1. Race Conditions

#### Example: Todo Reordering

Approaches:

1. Gap strategy (1000, 2000...)
2. Fractional indexing
3. Linked list
4. Optimistic locking (version column)

#### Tradeoffs

- Gaps → simple, but eventually rebalancing needed
- Fractional → avoids rebalance, precision issues
- Linked list → complex reads
- Optimistic locking → safe but retries required

---

### 2. Idempotency

Flow:

1. Client sends idempotency key
2. Server stores key + response
3. Retry returns cached response

Use cases:
- Payments
- Order creation

---

### 3. Distributed Locking

Options:
- Redis (SET NX)
- Zookeeper

Avoid when:
- You can use idempotency instead

---

### 4. Ordering Guarantees

- Kafka → partition-level ordering
- SQS → no strict ordering (unless FIFO)

Handling duplicates:
- Deduplication
- Idempotent consumers

---

## 🗄️ DAY 3 — Storage + Scaling + Caching

### 🎯 Goal
Design systems that scale cleanly.

---

### 1. Caching Strategies

- Cache Aside
- Write Through
- Write Behind

#### Problems

- Cache stampede → use request coalescing
- Stale data → TTL + invalidation
- Hot keys → sharding or replication

---

### 2. Consistent Hashing

Used for:
- Load balancing
- Distributed caches

Key ideas:
- Hash ring
- Virtual nodes

---

### 3. Database Scaling

- Read replicas
- Sharding (by user_id)
- Partitioning

#### Risks

- Hot partitions
- Cross-shard queries

---

### 4. Tradeoff Example

"Sharding by user_id improves locality but risks celebrity hot spots."

---

## 📡 DAY 4 — Messaging + Async Systems

### 🎯 Goal
Master event-driven architectures.

---

### 1. Queue Systems

| System | Strength |
|------|--------|
| Kafka | replay, ordering |
| SQS | simplicity |
| Pub/Sub | fanout |

---

### 2. Delivery Guarantees

- At-most-once
- At-least-once (most common)
- Exactly-once (rare, expensive)

---

### 3. Failure Handling

- Retries (exponential backoff)
- Dead letter queues
- Circuit breakers

---

### 4. Event-Driven Design

Examples:
- Notification system
- Activity feed

---

## 🧠 DAY 5 — Full Mock Interviews

### 🎯 Goal
Simulate real interviews.

---

### Practice Systems

- Chat system
- News feed
- Rate limiter
- File upload system
- Search system

---

### For Each Design

Cover:
- APIs
- Data model
- Scaling
- Tradeoffs
- Failure modes

---

## 🔥 Senior-Level Signals

Always include:

### 1. Monitoring

- Latency
- Error rates
- Throughput

---

### 2. SLOs + Rollback

Example:
"If latency exceeds 200ms, rollback caching layer"

---

### 3. Security

- Authentication
- Authorization
- Rate limiting

---

## 🧪 Practice Method

1. Pick a system
2. Talk out loud
3. Time yourself (30–40 min)
4. Record and refine

---

## 🚀 Final Notes

### Weak Signals

- Jumping into implementation too early
- No tradeoffs
- Overengineering

---

### Strong Signals

- Clear structure
- Tradeoffs early
- Handling edge cases proactively

---

## 🧠 Rapid Fire Topics

- Idempotency
- Cursor pagination
- Cache invalidation
- Hot partitions
- Eventual consistency
- WebSockets vs polling
- Rate limiting

---

## 🏁 Outcome

By the end of 5 days, you should be able to:

- Lead system design discussions
- Defend architectural decisions
- Identify scaling and failure risks
- Communicate like a senior/staff engineer

---

If needed, extend this plan with mock interviews and deeper dives into weak areas.


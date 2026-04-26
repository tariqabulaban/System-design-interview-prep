Consistency and CAP for Senior and Staff-Level Interviews

Consistency is one of the most important and misunderstood topics in system design. At senior and staff level, the goal is not just to name CAP theorem. The goal is to explain what kind of consistency a product actually needs, what tradeoffs are being made, and where weaker consistency is acceptable.

1. What consistency means

Consistency describes what a client is allowed to observe after data changes.

Important question:

- after a write happens, when and where will readers see that write?

2. Common kinds of consistency

Strong consistency:

- once a write is acknowledged, all subsequent reads see the latest value

Eventual consistency:

- replicas may temporarily disagree, but they converge over time

Read-after-write consistency:

- a client that writes data can immediately read its own update

Monotonic reads:

- once a client has seen a newer value, it should not later see an older one

Why this matters:

- different products need different guarantees

3. Real examples

Strong consistency is important for:

- account balances
- inventory reservations
- permission changes

Eventual consistency is often acceptable for:

- social feed counts
- analytics dashboards
- product recommendations
- search indexing

4. Why weaker consistency appears

Distributed systems replicate data for:

- higher availability
- lower latency
- fault tolerance

Replication creates the chance that different nodes temporarily see different values.

5. What CAP theorem says

CAP refers to:

- Consistency
- Availability
- Partition tolerance

Partition tolerance means:

- the system keeps operating even when network communication between nodes is partially broken

The key takeaway:

- when a network partition happens, a distributed system usually must choose between stronger consistency and higher availability

Senior/staff-level point:

- CAP is about behavior during partitions, not a general label for a system at all times

6. Consistency vs availability during partitions

Choose consistency:

- reject or delay some requests to avoid stale or conflicting results

Choose availability:

- continue serving requests even if some nodes are out of sync

Real example:

- a banking transfer system often prefers consistency
- a social likes counter often prefers availability

7. Quorums

Some systems use quorum reads and writes.

Write quorum:

- enough replicas must acknowledge a write before it succeeds

Read quorum:

- enough replicas are read so the client likely sees the newest value

Why this matters:

- quorum settings let systems tune the balance between consistency, latency, and availability

8. Stale reads

A stale read happens when the client sees old data even though a newer value exists somewhere else.

This is often acceptable for:

- feeds
- dashboards
- caches

This is often dangerous for:

- account balances
- inventory
- permissions or security decisions

9. Conflict resolution

If different replicas accept writes independently, conflicts may happen.

Common approaches:

- last-write-wins
- merge logic
- application-level resolution
- CRDT-style designs in specialized systems

Why this matters:

- some data types tolerate conflict better than others

10. Common mistakes

- saying "eventual consistency is fine" without checking business impact
- quoting CAP without tying it to the actual failure scenario
- assuming all stale reads are harmless
- treating consistency as only a database concern instead of an end-to-end product concern

11. Real examples

Need stronger consistency for:

- payments
- checkout inventory
- role and permission changes

Can often use weaker consistency for:

- notifications
- recommendation feeds
- social counters
- search indexes

12. Staff-level discussion prompts

- What user-visible inconsistency is acceptable?
- Can stale reads cause money loss or security problems?
- Are we optimizing for availability during partitions?
- Do we need read-after-write consistency?
- Where does the system intentionally allow lag?

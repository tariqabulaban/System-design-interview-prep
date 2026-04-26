Replication and Partitioning for Senior and Staff-Level Interviews

Replication and partitioning are core scaling and availability techniques in distributed systems. At senior and staff level, the key is understanding why data is copied, why data is split, and what correctness and operational complexity each decision introduces.

1. What replication is

Replication means storing copies of data on multiple nodes.

Why replication exists:

- improve availability
- protect against node failure
- improve read scale
- reduce geographic latency

2. Common replication models

Primary-replica:

- one primary handles writes
- replicas copy data from the primary

Multi-primary:

- more than one node accepts writes
- useful in some architectures, but introduces conflict complexity

Leaderless replication:

- multiple nodes may accept writes without one permanent leader

Why this matters:

- each model makes different tradeoffs in consistency, write complexity, and failure handling

3. Primary-replica replication

This is the most common mental model in interviews.

How it works:

- write goes to the primary
- replicas copy the update
- reads may go to either the primary or replicas depending on consistency needs

Benefits:

- simpler write path
- easy read scaling

Downsides:

- primary can become a bottleneck
- replicas may lag
- failover needs care

4. Replication lag

Replication lag means replicas are behind the primary.

Why this matters:

- a user may write data, then read from a replica and not see the update yet

Real example:

- after changing account settings, the user may see the old value briefly if reads go to a lagging replica

5. What partitioning is

Partitioning means splitting data into subsets so not all data lives on one node.

Another common word is sharding.

Why partitioning exists:

- increase write scale
- increase total storage capacity
- reduce contention on one machine

6. Good partition keys

A partition key decides where data lives.

Good partition keys:

- spread traffic fairly evenly
- align with major query patterns
- avoid concentrating all activity on one node

Examples:

- `tenant_id`
- `user_id`
- `device_id`

Bad partition keys:

- values with extreme skew
- keys that force many cross-partition reads

7. Hot partitions

A hot partition happens when one partition receives much more traffic than others.

Why it matters:

- one shard becomes overloaded even if the rest of the cluster is fine

Real example:

- one celebrity account creates a hot user partition in a social system

8. Repartitioning and resharding

Traffic and data growth change over time, so partitioning is not a one-time design decision.

Resharding means:

- moving data as the cluster grows or the key distribution changes

Why this is hard:

- data movement is expensive
- application routing may need changes
- hot traffic may continue during migration

9. Replication vs partitioning

Replication:

- copies the same data to multiple nodes
- helps availability and read scale

Partitioning:

- splits different subsets of data across nodes
- helps write scale and storage scale

Most large systems use both.

10. Common tradeoffs

Replication helps:

- durability
- availability
- read throughput

But it also introduces:

- lag
- failover complexity
- consistency tradeoffs

Partitioning helps:

- write throughput
- storage scale

But it also introduces:

- cross-shard query difficulty
- operational complexity
- hot key risk

11. Real examples

Use replication for:

- scaling read-heavy databases
- keeping standby copies for failover
- lowering read latency across regions

Use partitioning for:

- large event streams
- very large tenant-based SaaS systems
- high-write messaging or telemetry platforms

12. Staff-level discussion prompts

- Are we scaling for reads, writes, or both?
- Can the product tolerate replica lag?
- What is the partition key and why?
- What happens if one partition becomes hot?
- How hard will resharding be later?
- Do clients ever need cross-partition transactions or queries?

Relational Database Design for Senior and Staff-Level Interviews

This document is meant to be a real study guide, not just a list of prompts. At senior and staff level, you are expected to explain not only what a relational database is good at, but also how to model data safely, how to scale it, how to evolve it over time, and what tradeoffs you are making.

What interviewers are usually testing:

- Can you model entities and relationships in a clean, maintainable way?
- Can you choose keys, constraints, and indexes based on real access patterns?
- Can you explain normalization, denormalization, and why you chose one over the other?
- Can you reason about transactions, isolation, and concurrent updates?
- Can you explain how the database will behave in production under scale, failure, and schema change?

1. When to use a relational database

Relational databases are a strong default choice when the domain is structured and correctness matters.

Use a relational database when:

- Data has clear structure and well-defined relationships
- You need to join across related entities
- You need strong data integrity guarantees
- You need transactions across multiple rows or tables
- The business domain has rules the database should help enforce

Typical use cases:

- Financial systems where balances and transfers must stay correct
- E-commerce systems with orders, line items, payments, and inventory
- User and organization models with memberships and permissions
- Internal systems with reporting, auditability, and predictable structure

Common technologies:

- PostgreSQL
- MySQL
- MariaDB
- Microsoft SQL Server
- Oracle Database
- SQLite for local, embedded, or lightweight use cases

Query language:

- SQL is the main language and supports `SELECT`, `INSERT`, `UPDATE`, `DELETE`, `JOIN`, aggregation, window functions, filtering, and ordering

Senior/staff-level tradeoff:

- Relational databases make correctness and ad hoc querying easier, but they can become harder to scale when write throughput is extremely high, the schema changes constantly, or the workload is naturally distributed across many nodes.

2. Core modeling concepts

Relational design starts with modeling the domain in terms of entities, attributes, and relationships.

Common entities:

- `users`
- `organizations`
- `projects`
- `orders`
- `payments`

Common relationship types:

- One-to-one: one user has one profile
- One-to-many: one organization has many users
- Many-to-many: many users belong to many groups

Example:

- One user can create many tags
- One organization can have many users
- Many users can belong to many groups through a join table such as `user_groups`

Key design principle:

- Model the business domain first, then validate that the schema supports the main query patterns. If you optimize too early around one endpoint, the schema often becomes brittle.

Common mistake:

- Designing tables only around one immediate API response. That can look efficient at first, but it often creates duplication, weak integrity, and painful future migrations.

3. Keys and identifiers

Every table should have a clear primary key. The primary key is how the database identifies each row uniquely, and that choice affects indexing, sharding strategy, and application logic.

Common primary key choices:

- Auto-increment integer
- UUID
- ULID
- Snowflake-style ID
- Natural key in rare cases

What each means:

- Auto-increment integer: small, fast, and index-friendly. Good in single-primary setups, but harder when IDs need to be generated across multiple distributed writers.
- UUID: globally unique and convenient in distributed systems. The downside is that UUIDs are larger and can be less cache- and index-friendly than integers.
- ULID or sortable IDs: useful when you want distributed uniqueness plus approximate time ordering.
- Natural key: something like an email or external account number. Usually risky as a primary key because business identifiers can change.

Other important key-related concepts:

- Foreign keys connect related tables and help enforce referential integrity.
- Unique constraints enforce business rules like "an email can only appear once."
- Composite keys are useful when uniqueness depends on more than one column.

Examples:

- `email` may need a unique constraint because duplicate emails break login and identity assumptions.
- `organization_id + slug` may need to be unique so a slug is unique within an organization but not necessarily globally.
- `user_groups(user_id, group_id)` often uses a composite primary key because the pair itself is the business identity.

4. Normalization vs denormalization

Normalization means splitting data into related tables so each fact is stored once in the right place. The goal is to reduce duplication and make updates safer.

Benefits of normalization:

- Less duplicated data
- Easier updates because one fact is changed in one place
- Stronger consistency and integrity
- Lower risk of update anomalies

Example:

- Instead of storing the user's email on every order row, store the user in `users` and reference `user_id` from `orders`.

Denormalization means intentionally duplicating data to make reads faster or simpler.

Benefits of denormalization:

- Faster read queries
- Fewer joins on hot paths
- Better performance for dashboards, feeds, or read-heavy views

Example:

- You may copy `organization_name` into a reporting table or materialized view if joins become too expensive for a critical read path.

Tradeoff:

- Normalization improves correctness but can increase query complexity.
- Denormalization improves read performance but creates consistency challenges because duplicated data must stay in sync.

Senior/staff-level lens:

- Start normalized unless you have clear evidence that read performance or product requirements justify denormalization.
- Denormalization should be a deliberate optimization backed by query patterns and measurement, not just a guess that "joins are slow."

5. Constraints and data integrity

One of the biggest strengths of relational databases is that they can enforce important business invariants directly in the schema.

Useful constraints:

- `PRIMARY KEY`: ensures each row is unique and identifiable
- `FOREIGN KEY`: ensures referenced rows actually exist
- `UNIQUE`: prevents duplicate values where business rules require uniqueness
- `NOT NULL`: ensures required columns are always populated
- `CHECK`: enforces custom rules like valid ranges or allowed states
- Default values: provide safe defaults when the application omits a column

Examples:

- A price should not be negative, so a `CHECK (price >= 0)` constraint can enforce that.
- A payment should not exist without an order, so `payments.order_id` should reference `orders.id`.
- A user's email should be unique to avoid ambiguous account ownership.

Why this matters:

- Application-layer validation is not enough in concurrent systems or multi-service architectures.
- Constraints protect the data even if a buggy service, background job, or migration script writes invalid records.

6. Indexing

Indexes are one of the most important topics in relational database design. They speed up lookups by creating an additional structure that helps the database find rows without scanning the entire table.

Why indexes exist:

- Without an index, the database may have to read every row in a table to find matches.
- With the right index, the database can jump directly to the relevant subset of rows.

Common index types:

- B-tree: the default and most common index type; good for equality lookups, ranges, and sorting
- Hash: mainly for equality lookups in some databases, but less commonly relied on in practice
- GIN: useful in PostgreSQL for full text search, arrays, and `JSONB`
- GiST: useful for geometric data, ranges, and specialized data types
- Composite indexes: indexes over multiple columns used together in a query
- Covering indexes: indexes that include enough columns for the database to satisfy a query without reading the base table, depending on database support

Examples:

- Index `email` for fast login lookups
- Index `(organization_id, created_at)` for listing recent records within one tenant
- Index `(user_id, status)` for queries like "give me all active objects for this user"

Important design rule:

- Add indexes based on actual query patterns. Indexing every column is not good design because indexes cost memory, storage, and write performance.

Tradeoffs:

- More indexes improve read latency
- More indexes slow inserts and updates because every relevant index must also be updated
- Large indexes increase storage use and cache pressure

Important concept: left-prefix rule

- If you have an index on `(organization_id, created_at)`, it helps queries that filter by `organization_id`, and often queries that filter by both `organization_id` and `created_at`.
- It usually does not help much for queries that only filter by `created_at`, because the leading column is `organization_id`.

Senior/staff-level lens:

- You should be able to explain not only that an index is needed, but exactly which query it helps and why.
- You should also know when a full table scan is acceptable, such as on a small table or an infrequent analytical query.

7. Joins and query design

Relational databases are specifically designed to answer questions across related tables. Joins are a feature, not a smell.

Common join types:

- `INNER JOIN`: return rows that match on both sides
- `LEFT JOIN`: keep rows from the left side even if no match exists on the right
- `RIGHT JOIN`: the mirror of `LEFT JOIN`, though less commonly used
- `FULL OUTER JOIN`: return rows from both sides even when one side has no match, where supported

Good design questions:

- Are the join columns indexed?
- Are you joining a bounded set of rows or a huge unfiltered table?
- Does the query run on a latency-sensitive path?
- Should part of the result be materialized ahead of time?

Common mistake:

- Saying "joins are slow" without context. Poorly designed joins can be slow, but well-indexed joins on appropriately sized datasets are often exactly the right solution.

8. Transactions and ACID

Transactions let a group of database operations succeed or fail together. This is critical when partial success would leave the system in a bad state.

ACID stands for:

- Atomicity: either all operations in the transaction happen or none do
- Consistency: the transaction preserves valid database rules and invariants
- Isolation: concurrent transactions do not interfere in unsafe ways
- Durability: once committed, the data survives crashes

Use transactions when:

- Multiple writes must be treated as one logical unit
- Business rules span multiple rows or tables
- A partial write would create corruption, overcounting, or broken references

Examples:

- Creating an order and reserving inventory
- Recording a payment and updating a balance
- Moving money from one account to another

Senior/staff-level tradeoff:

- Transactions improve correctness, but large or long-running transactions hold locks longer, increase contention, and can reduce throughput.
- Staff-level answers often include how to keep transactions short and bounded.

9. Isolation levels and concurrency

Concurrency problems happen when multiple clients try to read and write the same data at the same time. Isolation levels define how much one transaction is protected from the effects of others.

Common isolation levels:

- Read uncommitted: allows very weak isolation and is rarely used in serious systems
- Read committed: a transaction only sees committed data; common default in many systems
- Repeatable read: repeated reads inside a transaction are more stable, though exact behavior differs by database
- Serializable: strongest isolation; behaves most like transactions happening one at a time, but can reduce throughput

Common anomalies:

- Dirty reads: reading data another transaction has not committed yet
- Non-repeatable reads: reading the same row twice and getting different results
- Phantom reads: rerunning a query and seeing new rows appear
- Lost updates: two writers overwrite each other unintentionally

Patterns to handle concurrency:

- Row-level locking when a row must not be updated concurrently
- `SELECT ... FOR UPDATE` to lock rows before a dependent write
- Optimistic locking with a version column when collisions are possible but uncommon
- Idempotent write patterns when retries are likely

Example scenario:

- Two customers try to buy the last item in stock.
- Without locking or optimistic conflict detection, both requests may succeed logically and inventory can go negative.

10. Schema evolution and migrations

Schema evolution matters because real systems change. Tables gain columns, constraints tighten, indexes are added, and data models evolve as the product evolves.

The phrase "prefer additive changes first" means this:

- Start with changes that do not immediately break old readers or old writers.
- Adding a new nullable column is usually safer than renaming or deleting an old one.
- Adding a new table is usually safer than repurposing an old table in place.

Why additive changes are safer:

- In real deployments, not all application instances update at the same time.
- Background jobs, scripts, analytics systems, and other services may still expect the old schema for a while.
- If the database change is backward-compatible, rollout is safer and easier to reverse.

A common safe migration sequence looks like this:

1. Add the new column or table in a backward-compatible way.
2. Update the application to start writing to both old and new structures if needed.
3. Backfill old records into the new shape.
4. Switch reads to the new field or table.
5. Enforce stronger constraints like `NOT NULL` only after data is complete and all writers are updated.
6. Remove the old column or path later, once you are sure nothing still depends on it.

What "backfill data before making fields required" means:

- Suppose you add a `country_code` column to `users`.
- If you immediately declare it `NOT NULL`, all existing rows will violate the schema unless you populate them first.
- A safer approach is to add the column as nullable, populate existing rows, update application code, and only then enforce `NOT NULL`.

What "avoid breaking reads and writes during rollout" means:

- If new code expects a column that old code does not populate yet, or if old code depends on a column you removed too early, production will fail mid-rollout.
- Safe schema changes assume mixed-version deployments for some period of time.

What "use multi-step migrations for large tables" means:

- Large tables can take a long time to rewrite or lock.
- A single big `ALTER TABLE` may block writes or create operational risk.
- Instead, break the change into smaller phases that avoid long blocking operations.

What "separate schema changes from application behavior changes" means:

- Do not combine a risky schema migration and a major logic change in one rollout if you can avoid it.
- If something fails, it becomes much harder to know whether the database change or the code change caused it.

Staff-level lens:

- Schema changes affect not just the application, but also ETL jobs, reporting, data science consumers, dashboards, and operational scripts.
- Safe migrations are as much about organizational coordination as SQL syntax.

11. Partitioning and scaling

Relational databases can scale very far, but they do not scale infinitely without design work.

Common scaling tools:

- Vertical scaling: add more CPU, RAM, or faster disks to one machine
- Read replicas: copy data to additional nodes for read traffic
- Table partitioning: split one logical table into physical partitions
- Sharding: split data across independent databases or clusters
- Caching: reduce repeated reads from the database
- Archival: move cold or historical data out of hot operational tables

What each means:

- Vertical scaling is the simplest path early on, but it has a ceiling.
- Read replicas help read-heavy workloads, but they may lag behind the primary and can break read-after-write assumptions.
- Partitioning helps large tables by making scans, maintenance, and retention more manageable.
- Sharding distributes load further but adds major application and operational complexity.

Partitioning styles:

- Range partitioning: split by ranges, such as date
- Hash partitioning: distribute rows evenly by a hash of one or more columns
- List partitioning: split by explicit value groups

Examples:

- Partition an events table by month if most queries are time-bounded
- Partition customer-heavy data by `organization_id` if many queries are tenant-scoped

Tradeoffs:

- Read replicas improve throughput, but replication lag can make data appear stale
- Sharding improves scale, but cross-shard joins, rebalancing, and transactions become much harder

12. Multi-tenancy

In B2B systems, data usually belongs to different customers or organizations. The database design must make that isolation explicit.

Common multi-tenant models:

- Shared database, shared schema with a `tenant_id` column
- Shared database, separate schema per tenant
- Separate database per tenant

What they mean:

- Shared schema is operationally simpler and easier to scale centrally, but every query and index strategy must respect tenant boundaries.
- Separate schemas improve logical separation, but schema management becomes more complex as tenant count grows.
- Separate databases improve isolation and can simplify compliance for some customers, but they increase cost and operational overhead significantly.

Key rule:

- Tenant scoping should not depend on the client remembering to pass the right filter. It should be built into query patterns, access controls, or both.

13. Replication, backup, and disaster recovery

Senior/staff engineers think about what happens when hardware fails, a region goes down, or a bad deploy corrupts data.

Replication concerns:

- Primary-replica replication: one node accepts writes and others replicate from it
- Asynchronous replication: faster, but replicas may lag and lose the latest data on failure
- Synchronous replication: stronger durability, but slower writes and more coordination cost
- Failover strategy: how the system promotes a replica if the primary dies
- Read-after-write consistency: whether a user can safely read from a replica immediately after writing to the primary

Backup and recovery concerns:

- Full backups: periodic snapshots of the whole database
- Incremental backups: capture only changes since the last backup
- Point-in-time recovery: replay logs to restore to a precise moment
- Recovery time objective: how quickly the service must recover
- Recovery point objective: how much data loss is acceptable

Good interview question to answer:

- If the primary dies right now, how much data can you afford to lose, and how long can the system be unavailable?

14. Observability and operations

A relational database is only "done" if the team can operate it safely in production.

Important metrics:

- Query latency: how long queries take
- Slow queries: queries that are outliers or gradually degrading
- Connection count: too many connections can overwhelm the database
- CPU and memory: signal compute pressure or cache effectiveness
- Disk usage and IOPS: common bottlenecks for storage-heavy workloads
- Lock wait time: signals concurrency pain
- Replication lag: signals stale replica risk
- Deadlocks: show conflicting write patterns
- Cache hit ratio: indicates whether the working set fits in memory

Operational tools and practices:

- `EXPLAIN` and query plans to understand how the database executes a query
- Slow query logs to find expensive operations
- Connection pooling so the app does not open too many direct connections
- Vacuum and autovacuum in PostgreSQL to reclaim space and maintain visibility maps
- Index maintenance and statistics refresh to keep the optimizer effective

15. Security and compliance

Database design also includes who can access data, how data is protected, and how sensitive records are handled.

Important topics:

- Encryption at rest so disk theft does not expose raw data
- Encryption in transit so data is protected between app and database
- Role-based database access so engineers and services get only the permissions they need
- Secret management for passwords, certificates, and connection strings
- Row-level security where supported
- Audit logging for sensitive reads and writes
- Data retention and deletion policies
- PII handling, masking, and field-level protections where needed

Examples:

- PostgreSQL row-level security can help enforce tenant-scoped access
- Payroll or medical tables may need more restrictive access than general account tables

16. Common relational technologies and what they are known for

PostgreSQL:

- Strong general-purpose relational database
- Excellent SQL support and rich extensions
- Strong indexing options, including advanced types
- `JSONB` support for semi-structured data
- Often the best default answer for modern systems

MySQL:

- Very common in web applications
- Large ecosystem and operational familiarity
- Strong choice when the team already knows it well

SQL Server:

- Strong enterprise integration, especially in Microsoft-heavy environments
- Mature tooling and reporting ecosystem

Oracle:

- Mature enterprise database used in large regulated and legacy environments
- Often chosen because of existing company investment and operational processes

SQLite:

- Lightweight embedded database with no separate server
- Great for local apps, mobile, testing, edge devices, and small workloads
- Not a typical answer for a large distributed backend, but useful to know

17. Languages, ORMs, and access patterns

The database is only part of the design. The application layer determines how safely and efficiently the team uses it.

Common languages:

- Python
- Java
- Kotlin
- Go
- C#
- JavaScript and TypeScript
- Ruby

Common libraries and ORMs:

- Python: SQLAlchemy, Django ORM
- Java/Kotlin: Hibernate, JPA, jOOQ
- Go: `database/sql`, GORM, sqlx
- C#: Entity Framework
- Node.js/TypeScript: Prisma, TypeORM, Knex
- Ruby: ActiveRecord

Senior/staff-level tradeoff:

- ORMs improve developer speed and reduce boilerplate for common CRUD operations.
- Raw SQL is often better for complex joins, reporting queries, performance-sensitive paths, and precise control.
- Strong teams know when to use the ORM and when to step below it.

18. Example schema

Example domain: organizations, users, tags, and groups

```sql
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name TEXT NOT NULL
);

CREATE TABLE users (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL REFERENCES organizations(id),
  email TEXT NOT NULL UNIQUE,
  created_at TIMESTAMP NOT NULL DEFAULT NOW()
);

CREATE TABLE tags (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL REFERENCES organizations(id),
  created_by UUID NOT NULL REFERENCES users(id),
  name TEXT NOT NULL,
  created_at TIMESTAMP NOT NULL DEFAULT NOW(),
  UNIQUE (organization_id, name)
);

CREATE TABLE groups (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL REFERENCES organizations(id),
  name TEXT NOT NULL,
  UNIQUE (organization_id, name)
);

CREATE TABLE user_groups (
  user_id UUID NOT NULL REFERENCES users(id),
  group_id UUID NOT NULL REFERENCES groups(id),
  PRIMARY KEY (user_id, group_id)
);
```

Why this schema is useful:

- `organization_id` appears on tenant-scoped tables so queries and indexes can enforce organization boundaries.
- `UNIQUE (organization_id, name)` allows duplicate tag names across organizations but not within the same organization.
- The `user_groups` join table models a many-to-many relationship cleanly.

Example indexes:

```sql
CREATE INDEX idx_tags_org_created_at ON tags (organization_id, created_at DESC);
CREATE INDEX idx_tags_created_by ON tags (created_by);
```

Why these indexes help:

- `idx_tags_org_created_at` helps list recent tags inside one organization, which is a common tenant-scoped read.
- `idx_tags_created_by` helps queries like "show me all tags created by this user."

19. Interview discussion prompts

- Why choose PostgreSQL over DynamoDB here?
- Would you normalize or denormalize this relationship, and why?
- What indexes would you add for the main read path?
- How would you prevent double-spending or overselling?
- When would you introduce sharding instead of scaling vertically?
- How would you migrate a large table without downtime?
- Are replicas safe for this read path, or do you need primary reads?

20. Sample answers to interview prompts

Why choose PostgreSQL over DynamoDB here?

PostgreSQL is usually the better choice when the domain has strong relationships, transaction correctness matters, and query patterns are not completely fixed ahead of time. It is a better fit when you need joins, foreign keys, constraints, and flexible querying across related entities. DynamoDB is usually a better fit when access patterns are highly predictable, low-latency key-based access matters most, and you are willing to model the data around partition keys and denormalized reads. A strong answer is that PostgreSQL optimizes for correctness and flexibility, while DynamoDB optimizes for scale and predictable access patterns.

Would you normalize or denormalize this relationship, and why?

The safest default is to start normalized. Normalization reduces duplication, makes updates easier to keep consistent, and lets the database enforce integrity more naturally. For example, keeping `users`, `organizations`, and `tags` in separate tables with foreign keys is usually the right starting point. I would denormalize only when a read path is hot enough that joins become a measurable bottleneck, the duplicated data changes infrequently, and I have a clear plan for keeping copies in sync. A strong answer is to start normalized for correctness, then denormalize deliberately for proven read-performance reasons.

What indexes would you add for the main read path?

Indexes should come directly from the most important queries. If the main read path is "list recent tags for one organization," then an index like `(organization_id, created_at DESC)` is a strong choice because it supports both filtering and ordering. If another main query is "find all tags created by a user," then an index on `(created_by)` may help. If users log in by email, then `email` should be uniquely indexed. The strongest answer ties each proposed index to a specific `WHERE`, `JOIN`, or `ORDER BY` pattern instead of listing indexes generically.

How would you prevent double-spending or overselling?

This is primarily a concurrency-control problem. In a relational system, I would usually use a transaction plus row-level locking or `SELECT ... FOR UPDATE` on the row representing the resource being consumed, such as an inventory count or account balance. Inside that transaction, I would re-check the current value, perform the update, and commit only if the invariant still holds. For request retries, especially payments, I would also use idempotency keys so the same logical operation is not processed twice. A strong answer mentions both concurrency control for simultaneous writes and idempotency for duplicate requests.

When would you introduce sharding instead of scaling vertically?

I would scale vertically first because it is operationally much simpler. Sharding becomes worth it when one database node can no longer handle the required write throughput, storage growth, or maintenance burden, and when replicas, caching, and partitioning are no longer enough. I would also look for a natural sharding key such as `tenant_id` or `organization_id`, because sharding is much safer when most traffic is already partitionable. A strong answer is that sharding is a later-stage scaling tool introduced only when simpler options are exhausted, because it makes joins, transactions, rebalancing, and debugging significantly harder.

How would you migrate a large table without downtime?

I would use a backward-compatible, multi-step migration. A common pattern is to add the new column or table safely, deploy code that can work with both old and new schema shapes, backfill existing data in batches, switch reads to the new structure, enforce stronger constraints only after backfill is complete, and remove the old path later. For a large table, the important point is avoiding long blocking operations and assuming mixed-version application deployments during rollout. A strong answer emphasizes additive schema changes, batch backfills, and gradual cutover rather than one big breaking migration.

Are replicas safe for this read path, or do you need primary reads?

The answer depends on whether replication lag is acceptable. Replicas are usually fine for read-heavy paths where slightly stale data is tolerable, such as dashboards, reporting, or non-critical browsing. Primary reads are safer when the feature requires read-after-write consistency or when stale data could create a correctness problem, such as inventory, balances, or permission checks right after a change. A strong answer says replicas are for scale when staleness is acceptable, and primary reads are for correctness-sensitive paths.

21. Design checklist

- Are entities and relationships modeled clearly?
- Are primary keys and unique constraints correct?
- Are foreign keys and integrity rules defined where useful?
- Are indexes aligned to real query patterns?
- Are transaction boundaries clear?
- Is concurrency behavior understood?
- Is tenant isolation explicit?
- Is schema evolution safe?
- Is the scaling plan realistic?
- Are backup, recovery, observability, and security considered?

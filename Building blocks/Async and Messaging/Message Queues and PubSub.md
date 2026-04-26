Message Queues and Pub/Sub for Senior and Staff-Level Interviews

Message queues and pub/sub systems are fundamental building blocks for decoupling services, handling asynchronous work, and smoothing traffic spikes. At senior and staff level, the key skill is understanding what problem they solve, when to use a queue versus pub/sub, and what delivery guarantees actually mean in practice.

1. What a message queue is

A message queue lets one component send work to another component asynchronously.

The producer:

- creates a message and puts it on the queue

The consumer:

- reads the message later and processes it

Why this matters:

- the producer does not have to wait for the work to finish immediately

2. What pub/sub is

Publish/subscribe means a producer publishes an event, and multiple subscribers can receive it.

The publisher:

- emits an event like "order_created"

The subscribers:

- independently react to that event

Why this matters:

- pub/sub is great when many downstream systems care about the same event

3. Queue vs pub/sub

Queue:

- usually one consumer handles a given work item
- good for background jobs and task processing

Pub/sub:

- many consumers may each get the same event
- good for fan-out and event-driven architectures

Real examples:

- queue: image thumbnail generation after upload
- pub/sub: order created event consumed by billing, inventory, notifications, and analytics

4. Why asynchronous messaging exists

It helps with:

- decoupling services
- smoothing bursty traffic
- retrying failed work
- isolating slow or flaky downstream systems
- handling background jobs without blocking user requests

5. Important keywords

Producer:

- the component that sends a message

Consumer:

- the component that processes a message

Broker:

- the system that stores and routes messages, such as Kafka, RabbitMQ, or SQS

Topic:

- a named event stream used in pub/sub systems

Partition:

- a subdivision of a topic used for scale and parallelism

Offset:

- the logical position of a message in an ordered stream

Dead-letter queue:

- a place where repeatedly failing messages go for investigation or reprocessing

6. When to use a queue

Use a queue when:

- work does not need to happen inline with the user request
- only one worker should process each task
- retries are needed
- throughput spikes need smoothing

Real examples:

- sending emails
- image processing
- PDF generation
- async billing jobs
- webhook delivery retries

7. When to use pub/sub

Use pub/sub when:

- many downstream consumers need the same event
- the producer should not know all consumers
- the architecture is event-driven

Real examples:

- user signed up event
- order created event
- payment failed event
- inventory low event

8. Delivery guarantees

At-most-once:

- a message is delivered zero or one time
- simpler, but messages can be lost

At-least-once:

- a message is delivered one or more times
- common in practice
- consumers must handle duplicates safely

Exactly-once:

- often claimed, rarely free
- usually requires strong coordination and careful end-to-end system design

Senior/staff-level point:

- most real systems should be designed assuming at-least-once delivery and idempotent consumers

9. Ordering

Some systems preserve ordering only within a partition or queue, not globally.

Why this matters:

- if message order affects correctness, the partitioning strategy matters a lot

Real example:

- account events for one user may need ordering, but global ordering across all users is usually unnecessary and expensive

10. Retries and dead-letter queues

Retries are necessary because consumers fail.

Important questions:

- which failures should be retried?
- how long should backoff be?
- when should the message go to a dead-letter queue?

Dead-letter queues matter because:

- poison messages can otherwise block or repeatedly fail forever

11. Idempotency

Consumers should usually be idempotent.

Why:

- queues and event systems often redeliver messages

Example:

- if a payment event is processed twice, the consumer must avoid charging twice

Ways to do this:

- idempotency keys
- deduplication tables
- unique constraints

12. Common technologies

Kafka:

- durable distributed event log
- strong for high-throughput event streaming, analytics pipelines, and service event streams

RabbitMQ:

- classic message broker
- strong for work queues, routing patterns, and task distribution

Amazon SQS:

- managed queue service
- simple and reliable for async jobs and decoupled processing

Amazon SNS:

- managed pub/sub service
- useful for broadcasting to multiple consumers

Google Pub/Sub:

- managed pub/sub service for event-driven architectures

Redis streams or pub/sub:

- useful for lighter-weight messaging patterns, though tradeoffs depend on durability needs

13. Real examples

Use queues for:

- background jobs
- email sending
- retries of slow external integrations
- async workflow steps

Use pub/sub for:

- domain events
- fan-out notifications
- analytics ingestion
- event-driven microservice communication

14. Staff-level discussion prompts

- Is this task synchronous or asynchronous?
- Should one consumer process the work or many?
- What delivery guarantee do we actually need?
- Does order matter?
- How will we retry safely?
- What is the dead-letter strategy?

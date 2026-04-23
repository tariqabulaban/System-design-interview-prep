RPC and gRPC for Senior and Staff-Level Interviews

RPC stands for Remote Procedure Call. The basic idea is that one service calls another service as if it were calling a local function, even though the work is happening over the network. gRPC is a modern, high-performance RPC framework built by Google that usually runs over HTTP/2 and uses Protocol Buffers for strongly typed message schemas.

At senior and staff level, the important question is not just "what is gRPC?" It is when RPC is the right abstraction, what you gain from gRPC, and what complexity it introduces.

1. What RPC is

RPC is a communication style where the client invokes a named remote method.

Example mental model:

- `CreateUser(request)`
- `GetOrder(orderId)`
- `ReserveInventory(request)`

Instead of thinking in terms of resources and HTTP verbs, RPC thinks in terms of methods or procedures.

2. What gRPC is

gRPC is a specific RPC framework with a few major characteristics:

- strongly typed schemas using Protocol Buffers
- code generation for clients and servers
- efficient binary serialization
- HTTP/2 transport support
- support for unary and streaming calls

Important keyword: Protocol Buffers

- Protocol Buffers, or Protobuf, are an interface definition language and binary serialization format.
- You define request and response types in `.proto` files.
- Code generators create typed client and server stubs in many languages.

3. When to use RPC or gRPC

RPC or gRPC is usually a strong fit when:

- communication is internal service-to-service
- performance matters
- low latency matters
- teams want strongly typed contracts
- the same API will be used across many backend services
- client and server code generation is valuable

Real examples:

- microservices inside a company backend
- inventory service talking to order service
- auth service talking to user service
- recommendation service talking to feature services
- internal ML inference calls where latency matters

4. RPC vs REST mindset

REST models resources:

- `GET /users/123`

RPC models operations:

- `GetUser(GetUserRequest)`

Neither is automatically better. The right choice depends on who the clients are and what the communication pattern is.

REST is usually better for:

- public APIs
- browser-friendly external integrations
- resource-oriented CRUD APIs

gRPC is usually better for:

- internal APIs
- strongly typed service contracts
- high-throughput low-latency communication

5. gRPC communication patterns

Unary:

- one request, one response
- Example: `GetUser`

Server streaming:

- one request, many responses streamed back
- Example: subscribe to a stream of logs or results

Client streaming:

- many requests streamed from client, one final response
- Example: uploading a stream of telemetry batches

Bidirectional streaming:

- both sides stream messages independently over one connection
- Example: live synchronization or chat-like coordination between services

Why this matters:

- gRPC supports more than just request-response. Streaming is one of its strongest features.

6. HTTP/2 and why it matters

gRPC commonly uses HTTP/2.

Important keyword: multiplexing

- Multiplexing means multiple requests can share one TCP connection at the same time.
- This reduces connection overhead and can improve throughput for chatty service-to-service communication.

Important keyword: binary framing

- HTTP/2 frames messages in a binary format rather than plain text.
- This helps efficiency, though it is less human-readable than plain HTTP/1.1 JSON APIs.

7. Strengths of gRPC

- strongly typed schemas
- generated clients and servers
- efficient binary payloads
- good performance
- built-in streaming support
- good fit for polyglot backend environments

Why this matters:

- When many services communicate frequently, typed contracts and generated clients reduce integration bugs and hand-written boilerplate.

8. Weaknesses of gRPC

- harder to inspect manually than JSON over HTTP
- less browser-friendly without extra tooling
- operational debugging can be less intuitive for teams unfamiliar with it
- schema management and versioning still require discipline

Senior/staff-level point:

- gRPC improves performance and contract quality, but can increase tooling and debugging complexity.

9. Schema evolution and compatibility

In gRPC, the `.proto` schema is part of the contract.

Important keyword: backward compatibility

- A newer service version should usually still understand messages sent by older clients, and vice versa when possible.

Common rules:

- add new fields instead of reusing old field numbers
- do not change field meanings casually
- reserve removed fields so they are not accidentally reused

Why this matters:

- gRPC is strongly typed, so careless schema changes can break many clients at once.

10. Error handling

RPC systems need a structured way to return failures.

In gRPC, errors are usually communicated through status codes and optional metadata.

Common examples:

- invalid argument
- unauthenticated
- permission denied
- not found
- unavailable
- deadline exceeded

Important keyword: deadline

- A deadline is the maximum time the caller is willing to wait for the response.
- This is important in distributed systems so calls do not hang forever and cause cascading failures.

11. Retries, timeouts, and resilience

Internal RPC calls must be designed with failure in mind.

Important concepts:

- timeout: how long the caller waits
- retry: whether the caller should try again
- idempotency: whether retrying is safe
- circuit breaker: stopping repeated calls to an unhealthy dependency

Real example:

- If the order service calls inventory with gRPC, the call should usually have a deadline and only retry if the method is safe to retry.

12. Security

Common security concerns:

- TLS for encryption in transit
- service-to-service authentication
- authorization between services
- identity propagation for end-user context where needed

Real example:

- If an API gateway authenticates the user and then calls internal gRPC services, the downstream services may need the user identity or service identity in metadata.

13. Languages and ecosystem

One of gRPC's biggest strengths is multi-language support.

Common languages:

- Go
- Java
- Kotlin
- Python
- C#
- Node.js and TypeScript
- Ruby

Why this matters:

- Different teams can use different languages while still sharing the same `.proto` contract.

14. When gRPC is a bad fit

Do not default to gRPC when:

- the API is public-facing and should be easy for third parties to call
- browser compatibility without extra layers is important
- the workload is simple CRUD where REST is easier and more familiar
- the team lacks the tooling maturity to manage `.proto` contracts and generated code well

15. Real examples

Use gRPC for:

- internal microservice communication
- auth, payments, inventory, recommendation, and feature services talking internally
- streaming telemetry between backend systems
- high-QPS low-latency backend calls

Do not default to gRPC for:

- public developer APIs
- browser-only frontend APIs without a gateway
- simple external integrations where JSON over HTTP is enough

16. Example service diagram

Here is a realistic internal microservice example:

```text
+----------------+        gRPC         +-------------------+
| Order Service  | ------------------> | Inventory Service |
+----------------+                     +-------------------+
        |                                        |
        | ReserveInventory()                     | checks stock
        | GetInventory()                         | updates reservation state
        | StreamLowStockAlerts()                 | emits low-stock events
        v                                        v
  Creates order                           Reads/writes inventory store
```

Why this is a good gRPC example:

- the communication is internal
- the contract can be strongly typed
- the calls are frequent and latency-sensitive
- both unary calls and streaming calls make sense

17. Example `.proto` file

Below is a realistic `.proto` definition for an inventory service.

```proto
syntax = "proto3";

package inventory.v1;

service InventoryService {
  rpc GetInventory(GetInventoryRequest) returns (GetInventoryResponse);
  rpc ReserveInventory(ReserveInventoryRequest) returns (ReserveInventoryResponse);
  rpc ReleaseInventory(ReleaseInventoryRequest) returns (ReleaseInventoryResponse);
  rpc StreamLowStockAlerts(StreamLowStockAlertsRequest) returns (stream LowStockAlert);
}

message GetInventoryRequest {
  string sku = 1;
  string warehouse_id = 2;
}

message GetInventoryResponse {
  string sku = 1;
  string warehouse_id = 2;
  int32 available_quantity = 3;
  int32 reserved_quantity = 4;
  int32 reorder_threshold = 5;
}

message ReserveInventoryRequest {
  string request_id = 1;
  string order_id = 2;
  string sku = 3;
  string warehouse_id = 4;
  int32 quantity = 5;
}

message ReserveInventoryResponse {
  string reservation_id = 1;
  string order_id = 2;
  string status = 3;
  int32 reserved_quantity = 4;
}

message ReleaseInventoryRequest {
  string reservation_id = 1;
  string reason = 2;
}

message ReleaseInventoryResponse {
  string reservation_id = 1;
  string status = 2;
}

message StreamLowStockAlertsRequest {
  string warehouse_id = 1;
}

message LowStockAlert {
  string sku = 1;
  string warehouse_id = 2;
  int32 available_quantity = 3;
  int32 reorder_threshold = 4;
  string triggered_at = 5;
}
```

Why this `.proto` is realistic:

- `GetInventory` is a unary read
- `ReserveInventory` is a unary write used by the order flow
- `ReleaseInventory` lets another service undo a reservation
- `StreamLowStockAlerts` shows a server-streaming use case
- `request_id` can help support idempotency and tracing

18. Example gRPC calls

Unary call example: `GetInventory`

Request:

```json
{
  "sku": "sku_123",
  "warehouse_id": "wh_us_west_1"
}
```

Response:

```json
{
  "sku": "sku_123",
  "warehouse_id": "wh_us_west_1",
  "available_quantity": 42,
  "reserved_quantity": 8,
  "reorder_threshold": 10
}
```

Unary call example: `ReserveInventory`

Request:

```json
{
  "request_id": "req_abc_123",
  "order_id": "order_456",
  "sku": "sku_123",
  "warehouse_id": "wh_us_west_1",
  "quantity": 2
}
```

Response:

```json
{
  "reservation_id": "res_789",
  "order_id": "order_456",
  "status": "RESERVED",
  "reserved_quantity": 2
}
```

Server streaming example: `StreamLowStockAlerts`

Request:

```json
{
  "warehouse_id": "wh_us_west_1"
}
```

Streamed responses:

```json
{
  "sku": "sku_123",
  "warehouse_id": "wh_us_west_1",
  "available_quantity": 9,
  "reorder_threshold": 10,
  "triggered_at": "2026-04-22T18:10:00Z"
}
```

```json
{
  "sku": "sku_777",
  "warehouse_id": "wh_us_west_1",
  "available_quantity": 4,
  "reorder_threshold": 8,
  "triggered_at": "2026-04-22T18:11:00Z"
}
```

19. What generated code gives you

From the `.proto` file, gRPC tooling typically generates:

- request and response classes
- a typed client stub
- a server interface or abstract base

That means application code often looks conceptually like:

```text
inventoryClient.GetInventory(...)
inventoryClient.ReserveInventory(...)
inventoryClient.StreamLowStockAlerts(...)
```

This is one reason gRPC is attractive for internal systems: engineers call remote services using typed method signatures instead of hand-building HTTP requests.

20. Staff-level discussion prompts

- Is this API internal-only or externally consumed?
- Do we need strong typing and generated clients?
- Will the call graph become highly chatty?
- Do we need streaming?
- How will we handle deadlines, retries, and idempotency?
- Do we have the tooling and observability maturity to operate gRPC well?

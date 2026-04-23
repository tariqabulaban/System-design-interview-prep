REST for Senior and Staff-Level Interviews

REST is one of the most common API communication styles for external-facing services. At senior and staff level, it is not enough to say "REST uses HTTP." You should be able to explain when REST is a good fit, how it models resources, what tradeoffs it makes, and where it stops being the best choice.

1. What REST is

REST stands for Representational State Transfer. In practice, when people say "REST API," they usually mean an HTTP API that models business concepts as resources and uses standard HTTP methods like `GET`, `POST`, `PUT`, `PATCH`, and `DELETE`.

The key idea is:

- the URL identifies the resource
- the HTTP method describes the action
- the response returns a representation of the resource, usually JSON

Example:

- `GET /api/v1/users/123`
- `POST /api/v1/orders`
- `PATCH /api/v1/tags/tag_123`

2. When to use REST

REST is usually a strong fit when:

- you are building public or partner-facing APIs
- clients include browsers, mobile apps, third-party integrators, or internal teams
- the domain maps well to resources like users, orders, payments, comments, or tags
- caching, HTTP tooling, auth middleware, and debuggability matter
- human readability and ecosystem familiarity are important

Real examples:

- e-commerce order APIs
- user account management APIs
- payment or billing APIs
- content management APIs
- admin panel backends

3. Core concepts

Resource:

- A resource is the thing being modeled, such as a user, order, invoice, or message.

Representation:

- A representation is how that resource is returned to the client, usually JSON.

Statelessness:

- Each request should contain the information needed to process it. The server should not rely on hidden per-client session state between calls unless you are explicitly using session-based auth.

Uniform interface:

- Clients interact with resources using a consistent interface, mainly HTTP methods, paths, headers, and response codes.

Why this matters:

- REST becomes easier to learn and operate when similar resources follow the same conventions.

4. HTTP methods and what they mean

- `GET`: read data without changing server state
- `POST`: create a resource or trigger a non-idempotent action
- `PUT`: replace a resource completely
- `PATCH`: partially update a resource
- `DELETE`: delete a resource

Important keyword: idempotent

- An operation is idempotent if sending it multiple times has the same effect as sending it once.
- `GET`, `PUT`, and `DELETE` are often designed to be idempotent.
- `POST` usually is not unless you add an idempotency mechanism.

5. Resource-oriented design

Good REST APIs usually model nouns, not verbs.

Good examples:

- `/users`
- `/users/{userId}`
- `/orders/{orderId}/items`

Less ideal examples:

- `/createUser`
- `/deleteOrder`

Why:

- REST works best when the path represents the thing and the method represents the action.

That said, action-style endpoints are sometimes still reasonable when the operation is not natural CRUD, such as:

- `POST /api/v1/jobs/{jobId}:cancel`
- `POST /api/v1/tags/{tagId}:archive`

6. Status codes

REST relies heavily on HTTP semantics, so status codes matter.

Common ones:

- `200 OK`: successful read or update
- `201 Created`: successful creation
- `202 Accepted`: accepted for asynchronous processing
- `204 No Content`: successful delete or update with no body
- `400 Bad Request`: invalid input
- `401 Unauthorized`: missing or invalid auth
- `403 Forbidden`: authenticated but not allowed
- `404 Not Found`: resource does not exist
- `409 Conflict`: conflict such as duplicate creation or stale update
- `429 Too Many Requests`: rate limit hit
- `500 Internal Server Error`: unexpected server failure

Senior/staff-level point:

- Correct status codes improve client behavior, observability, and retry logic.

7. Headers and metadata

Important headers:

- `Authorization`
- `Content-Type`
- `Accept`
- `Idempotency-Key`
- `If-Match`
- cache-related headers like `ETag` and `Cache-Control`

Why these matter:

- headers carry auth, content negotiation, retry safety, concurrency control, and caching behavior

8. Pagination, filtering, and sorting

REST list endpoints often need query parameters.

Examples:

- `GET /api/v1/orders?customerId=123&limit=20`
- `GET /api/v1/tags?createdBy=user_1&cursor=abc123`
- `GET /api/v1/products?category=books&sort=price`

Important keyword: cursor pagination

- Cursor pagination returns a token for the next page instead of a numeric offset.
- It is often better for large, changing datasets because offset pagination becomes unstable and slower as data grows.

9. Versioning

REST APIs often need a versioning strategy.

Common approaches:

- URL versioning: `/api/v1/...`
- header-based versioning

Why versioning matters:

- APIs live a long time
- clients upgrade slowly
- breaking changes need a safe rollout path

10. Strengths of REST

- widely understood
- easy to debug with normal HTTP tools
- good browser and proxy support
- works well with caching and auth middleware
- strong ecosystem support
- good fit for public APIs

11. Weaknesses of REST

- can be chatty if clients need many related resources
- can lead to over-fetching or under-fetching
- not ideal for strongly typed internal service-to-service contracts compared to gRPC
- not ideal for real-time bidirectional communication compared to WebSockets

12. When REST is better than other communication styles

REST is usually better than gRPC when:

- external clients need simplicity
- browser compatibility matters
- third-party developers will integrate with it

REST is usually better than WebSockets when:

- the workload is request-response
- clients do not need a persistent live connection

REST is usually better than SSE when:

- communication is not one-way streaming from server to client

13. Common interview tradeoffs

REST vs gRPC:

- REST is simpler and more universal
- gRPC is often better for internal high-performance typed service communication

REST vs WebSockets:

- REST is better for standard CRUD and stateless request-response
- WebSockets are better for interactive real-time bidirectional communication

REST vs SSE:

- REST is better for pull-based APIs
- SSE is better when the server needs to continuously push updates to the client

14. Real examples

Use REST for:

- account settings APIs
- product catalog APIs
- order management
- payments and billing APIs
- admin tools
- partner and public APIs

Do not default to REST for:

- multiplayer game state
- collaborative editing presence
- low-latency internal RPC between many microservices
- server-push live feeds without polling

15. Staff-level discussion prompts

- Would this API be public-facing or internal-only?
- Do clients need strong typing and generated SDKs?
- Is the workload mostly CRUD or event streaming?
- Will this become chatty for mobile clients?
- Do we need caching at CDN or proxy layers?
- How will we version the API without breaking clients?

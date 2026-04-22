API Design for Senior and Staff-Level Interviews

This document is no longer just a list of endpoint basics. At senior and staff level, the expectation is that you can design APIs that are clear, consistent, safe to evolve, and reliable under real production conditions.

What interviewers are looking for:

- Can you model resources cleanly?
- Can you explain tradeoffs instead of only naming best practices?
- Can you reason about failures, retries, permissions, and scale?
- Can you keep the API stable while the system evolves underneath it?
- Can you design something that multiple teams can use without constant coordination?

1. Start with the resource model

Good API design starts with clear domain modeling.

- Prefer nouns over verbs.
- Make the URL structure reflect the resources in the system.
- Keep naming consistent across endpoints.
- Avoid exposing internal implementation details in the API shape.

Examples:

- `GET /api/v1/tags`
- `GET /api/v1/tags/{tagId}`
- `POST /api/v1/tags`
- `PATCH /api/v1/tags/{tagId}`
- `DELETE /api/v1/tags/{tagId}`

Senior/staff-level tradeoff:

- A clean resource model reduces client confusion, makes documentation easier, and lowers the chance that different teams invent incompatible patterns.
- Sometimes action-style endpoints are still appropriate for workflows that are not natural CRUD operations, such as `POST /api/v1/tags/{tagId}:archive` or `POST /api/v1/jobs/{jobId}:cancel`.

2. Path variables vs query parameters vs request body

- Use a path variable when identifying a specific resource.
  Example: `/api/v1/tags/{tagId}`
- Use query parameters for filtering, sorting, searching, and pagination.
  Example: `/api/v1/tags?createdBy=tariqabulaban&limit=25`
- Use a request body when sending structured data to create or update a resource.

Design principle:

- Paths should identify "what resource"
- Query parameters should describe "which subset" or "how to view it"
- Request bodies should describe "what data to write"

Common mistake:

- Putting important mutable business data into query parameters instead of the request body.

3. HTTP methods and semantics

- `GET`: Read data. Should not change server state.
- `POST`: Create a new resource or trigger a non-idempotent action.
- `PUT`: Replace a resource completely.
- `PATCH`: Partially update a resource.
- `DELETE`: Remove a resource.

Senior/staff-level tradeoff:

- `PUT` is easier to reason about for full replacement, but can become risky when clients do not send all fields correctly.
- `PATCH` is often better for modern APIs because it reduces payload size and avoids accidental overwrites, but it makes merge behavior and validation more subtle.
- `POST` is frequently overused. Use it intentionally when the operation is truly create-like or action-like.

4. Headers and request metadata

- `Authorization: Bearer <token>` for auth
- `Content-Type: application/json` for JSON request bodies
- `Idempotency-Key: <unique-key>` for safe retries on non-idempotent writes
- `If-Match` or entity tags for optimistic concurrency control
- Cookies are more common for browser session-based auth

Why this matters:

- At scale, metadata is often as important as the request body.
- Headers help with authentication, retries, concurrency, tracing, and caching.

5. Authentication, authorization, permissions, and admin groups

Authentication answers "who is making the request?" Authorization answers "what are they allowed to do?"

- A user may be authenticated but still blocked from a resource.
- Keep roles simple. Common examples are `user`, `admin`, and `super_admin`.
- For fine-grained access, use permissions such as `tag:read`, `tag:create`, `tag:update`, and `tag:delete`.
- Admin groups are useful when multiple users need the same elevated access.
- The API must enforce authorization on the server side, even if the frontend also uses permission data for UX.

Example auth model:

```json
{
  "userId": "user_123",
  "tenantId": "tenant_abc",
  "groups": ["finance_admins"],
  "permissions": ["tag:read", "tag:create", "tag:update", "tag:delete"]
}
```

Recommended patterns:

- Return `401 Unauthorized` when the token is missing or invalid.
- Return `403 Forbidden` when the user is authenticated but lacks permission.
- Prefer group-based assignment for broad access management and permissions for fine-grained control.
- Scope access by tenant, team, or organization whenever possible.
- Design for least privilege by default.

Frontend vs backend permissions:

- The frontend may receive roles or permissions so it can hide buttons, disable actions, and avoid sending users down flows that will fail.
- The backend is still the source of truth and must always validate access.
- Frontend checks improve usability. Backend checks provide security.

Senior/staff-level tradeoff:

- Role-based access control is simpler to manage.
- Attribute-based or policy-based access control is more flexible, but harder to reason about and debug.
- Staff-level design often means knowing when simple RBAC is enough and when richer policies are worth the complexity.

6. Multi-tenancy and data isolation

This is one of the biggest gaps in many mid-level API discussions.

- If your system serves multiple customers or business units, isolation rules must be explicit.
- Tenant scoping should not depend only on the client sending the right filter.
- The backend should derive tenant context from auth or validated routing rules.

Examples:

- Good: infer `tenantId` from the authenticated session
- Risky: trust `tenantId` passed in the request body without validation

Questions to think through:

- Can one tenant ever access another tenant's data?
- Are admin users global or tenant-scoped?
- Are object IDs globally unique or only unique within a tenant?
- What happens to indexes and pagination when data is partitioned by tenant?

7. Status codes

- `200 OK`: Successful read or update
- `201 Created`: Resource successfully created
- `202 Accepted`: Request accepted for async processing
- `204 No Content`: Successful delete with no response body
- `400 Bad Request`: Invalid input
- `401 Unauthorized`: Missing or invalid authentication
- `403 Forbidden`: Authenticated but not allowed
- `404 Not Found`: Resource does not exist
- `409 Conflict`: Duplicate or conflicting request
- `412 Precondition Failed`: Optimistic locking or conditional request failed
- `429 Too Many Requests`: Rate limit hit
- `500 Internal Server Error`: Unexpected server failure
- `503 Service Unavailable`: Temporary overload or dependency outage

Senior/staff-level tradeoff:

- Precise status codes improve debuggability and client behavior.
- Overly generic `400` or `500` responses create operational blind spots and make retries harder to implement correctly.

8. Response design and contract consistency

Success responses should be predictable and easy for clients to parse.

Example:

```json
{
  "tagId": "tag_123",
  "status": "pending"
}
```

Error responses should follow a standard envelope.

Example:

```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "tagName is required",
    "details": [
      {
        "field": "tagName",
        "issue": "missing"
      }
    ],
    "requestId": "req_123"
  }
}
```

Best practices:

- Keep naming conventions consistent, usually `camelCase`.
- Include stable machine-readable error codes.
- Include a request or trace ID for debugging.
- Avoid returning raw internal exception messages.

9. Pagination and large-list design

Use pagination when returning lists of data so responses stay fast and predictable.

Example request:

- `GET /api/v1/tags?createdBy=tariqabulaban&cursor=abc123&limit=25`

Example response:

```json
{
  "items": [
    {
      "tagId": "tag_123",
      "tagName": "business_expense",
      "contentId": "JST-123",
      "createdBy": "tariqabulaban"
    }
  ],
  "nextCursor": "def456"
}
```

Tradeoffs:

- Offset pagination is simple and easy for humans, but becomes slow and inconsistent on large, changing datasets.
- Cursor pagination is more stable and scalable, but harder to debug manually and harder for arbitrary page jumps.

Senior/staff-level considerations:

- Define a default and max limit.
- Make sort order explicit.
- Ensure the cursor is derived from a stable indexed ordering.
- Think about what happens when data is inserted or deleted during pagination.

10. Idempotency and retries

- An operation is idempotent if sending the same request multiple times has the same effect as sending it once.
- `GET`, `PUT`, and `DELETE` are usually designed to be idempotent.
- `POST` is usually not idempotent unless you add an `Idempotency-Key`.

Example:

- A client times out while creating a tag and retries the request.
- With an `Idempotency-Key`, the server can return the original result instead of creating duplicates.

Why this matters:

- In distributed systems, clients retry.
- Load balancers retry.
- Background workers retry.
- If the API does not handle retries safely, duplicate writes become a production issue.

Staff-level lens:

- Idempotency is not just an API feature. It often requires storage design, deduplication windows, and replay semantics.

11. Concurrency control and write conflicts

Not every failure is a bug. Some are legitimate conflicting writes.

Patterns:

- Optimistic locking with version numbers
- Entity tags with `If-Match`
- Conflict detection with `409 Conflict`

Example scenario:

- Two admins edit the same tag at the same time.
- Without concurrency control, the last write silently overwrites the first.

Good design should answer:

- Do we allow last-write-wins?
- Do we reject stale updates?
- Do we merge partial updates?

12. Async APIs and long-running work

Not every request should block until all work is done.

Use async APIs when:

- Processing is slow
- Work fans out to other systems
- The action is handled by a queue or workflow engine

Common pattern:

- Client calls `POST /api/v1/tag-imports`
- API returns `202 Accepted`
- Response contains a `jobId` or status URL
- Client polls `GET /api/v1/tag-imports/{jobId}`

Example response:

```json
{
  "jobId": "job_123",
  "status": "pending"
}
```

Senior/staff-level tradeoff:

- Synchronous APIs are simpler.
- Asynchronous APIs are often more reliable and scalable for expensive operations, but they increase complexity for clients and observability.

13. Versioning and backward compatibility

Versioning is not just about `/v1`. It is about how safely the API evolves.

Common approaches:

- URI versioning: `/api/v1/tags`
- Header-based versioning
- Media-type versioning

Principles:

- Avoid breaking changes whenever possible.
- Prefer additive changes such as adding optional fields.
- Deprecate old fields before removing them.
- Give clients migration time and clear communication.

Breaking-change examples:

- Renaming fields
- Changing enum meanings
- Changing default sort order unexpectedly
- Tightening validation on existing clients without rollout planning

Staff-level lens:

- API design at scale includes migration strategy, client communication, and organizational discipline around compatibility.

14. Rate limiting, quotas, and abuse prevention

An API is not complete if it works only for well-behaved clients.

Consider:

- Per-user or per-tenant rate limits
- Burst vs steady-state traffic
- Expensive endpoints that need tighter limits
- Abuse detection and throttling

Useful responses:

- `429 Too Many Requests`
- Retry guidance in headers or documentation

15. Observability, auditability, and operability

Senior and staff engineers think about how the API behaves in production, not just on a whiteboard.

Key concerns:

- Request IDs and trace propagation
- Structured logs
- Metrics for latency, traffic, errors, and saturation
- Audit logs for security-sensitive actions
- Dashboards and alerts for critical endpoints

Examples:

- Deleting a tag may require an audit event recording who deleted it and when.
- A failing dependency may surface as elevated `503` rates on one API family.

16. Caching and performance

APIs often need to balance correctness, freshness, and latency.

Consider:

- Which reads are cacheable?
- Is stale data acceptable?
- Do clients need strong consistency or fast responses?

Useful tools:

- Cache headers
- ETags
- Conditional GETs
- Denormalized read models for expensive queries

Tradeoff:

- Better performance often increases complexity around invalidation and freshness guarantees.

17. Example API

Create a new tag

- `POST /api/v1/tags`
- Permission required: `tag:create`
- Tenant scope is derived from the authenticated user, not trusted from the body.

Request body:

```json
{
  "tagName": "business_expense",
  "createdBy": "tariqabulaban",
  "contentId": "JST-123"
}
```

Response: `201 Created`

```json
{
  "tagId": "tag_123",
  "tagName": "business_expense",
  "createdBy": "tariqabulaban",
  "contentId": "JST-123"
}
```

Get all tags created by Tariq

- `GET /api/v1/tags?createdBy=tariqabulaban&cursor=abc123&limit=25`
- Permission required: `tag:read`
- Regular users should only see tags they are allowed to access.
- Admins may be allowed to query across users within the same tenant.

Response: `200 OK`

```json
{
  "items": [
    {
      "tagId": "tag_123",
      "tagName": "business_expense"
    }
  ],
  "nextCursor": "def456"
}
```

Get one tag

- `GET /api/v1/tags/tag_123`
- Permission required: `tag:read`
- The caller must own the tag or belong to an allowed admin group.

Response: `200 OK`

```json
{
  "tagId": "tag_123",
  "tagName": "business_expense",
  "createdBy": "tariqabulaban",
  "contentId": "JST-123"
}
```

Update a tag name

- `PATCH /api/v1/tags/tag_123`
- Permission required: `tag:update`
- Only the owner, a team admin, or a platform admin can update it.
- For high-conflict objects, consider optimistic concurrency with `If-Match`.

Request body:

```json
{
  "tagName": "corporate_expense"
}
```

Response: `200 OK`

```json
{
  "tagId": "tag_123",
  "tagName": "corporate_expense"
}
```

Delete a tag

- `DELETE /api/v1/tags/tag_123`
- Permission required: `tag:delete`
- Restrict this to admins if deletion is sensitive.
- Consider audit logging for destructive actions.

Response: `204 No Content`

Admin-only endpoint example

- `POST /api/v1/admin/groups/finance_admins/members`
- Permission required: `group:manage`
- The actor should also be scoped to the correct tenant or admin domain.

Request body:

```json
{
  "userId": "user_456"
}
```

Response: `200 OK`

```json
{
  "groupId": "finance_admins",
  "userId": "user_456",
  "status": "added"
}
```

Async import example

- `POST /api/v1/tag-imports`
- Response: `202 Accepted`

```json
{
  "jobId": "job_123",
  "status": "pending"
}
```

18. Staff-level discussion prompts

These are the kinds of follow-up questions that move the conversation from senior to staff.

- How would you keep this API backward-compatible for five client teams?
- How would you migrate from role-based permissions to fine-grained permissions?
- How would you prevent duplicate writes during retries across regions?
- How would you support tenant isolation without making every query painfully slow?
- What metrics and audit logs would you require before launching this API?
- Which decisions would you standardize across the company versus leave to individual teams?

19. Design checklist

- Are resource names nouns instead of verbs?
- Are request and response shapes consistent?
- Are error responses standardized?
- Are status codes accurate and useful?
- Does the list endpoint support pagination at scale?
- Are retries safe where needed?
- Is authentication clearly defined?
- Are authorization rules clearly defined?
- Is tenant isolation explicit?
- Is concurrency behavior defined?
- Is backward compatibility planned?
- Are rate limits, observability, and audit logs considered?
- Can multiple teams adopt this API without inventing their own conventions?

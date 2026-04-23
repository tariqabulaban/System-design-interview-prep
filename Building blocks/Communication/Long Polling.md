Long Polling for Senior and Staff-Level Interviews

Long polling is a communication pattern where the client sends a request to the server, and the server keeps that request open until new data is available or a timeout is reached. Once the client receives a response, it immediately opens another request to wait for the next update.

Long polling is often described as a middle ground between simple polling and fully persistent real-time protocols like SSE or WebSockets.

1. What long polling is

In normal polling, the client asks the server for updates every few seconds whether anything changed or not.

In long polling:

- the client sends a request
- the server waits until something changes
- the server returns the new data or a timeout response
- the client immediately opens a new request

Important keyword: held request

- A held request is an HTTP request the server intentionally keeps open while waiting for an event.

2. Why long polling exists

Long polling exists because simple polling can be wasteful and slow.

Example:

- if a browser polls every 5 seconds, updates may be delayed by up to 5 seconds
- if the browser polls every 500ms, the server sees a lot of wasteful requests

Long polling improves freshness without requiring a fully bidirectional persistent connection.

3. When to use long polling

Long polling is a reasonable fit when:

- you need near-real-time updates
- the environment is based on standard HTTP request-response
- WebSockets or SSE are unavailable, unnecessary, or too operationally complex
- update frequency is moderate
- clients can tolerate reconnecting after each event or timeout

Real examples:

- chat or notification systems in older environments
- job status updates for browser clients
- support dashboards waiting for new tickets or events
- legacy enterprise systems where WebSockets are difficult to deploy

4. How long polling works

Typical flow:

1. client sends `GET /api/v1/notifications/long-poll`
2. server checks whether new data already exists
3. if data exists, server returns it immediately
4. if not, server keeps the request open for a limited time
5. when an event occurs, server returns the response
6. client immediately starts another request

If nothing happens before the timeout:

- the server returns an empty response, timeout response, or "no updates" payload
- the client reconnects and waits again

5. Strengths of long polling

- works over normal HTTP
- simpler than WebSockets in some environments
- can be easier to deploy through existing infrastructure
- better freshness than basic polling
- useful when the client only needs server-to-client updates

Why this matters:

- long polling is often a pragmatic step when teams want better real-time behavior without introducing a more stateful protocol

6. Weaknesses of long polling

- still creates a new HTTP request after every response
- not as efficient as SSE or WebSockets for sustained real-time traffic
- can create high connection churn
- server resources may be tied up waiting on many held requests
- latency can still be worse than true persistent streaming

Senior/staff-level point:

- long polling is often operationally simpler than WebSockets, but less efficient at scale because every update still involves a request lifecycle

7. Long polling vs regular polling

Regular polling:

- client asks repeatedly on a fixed interval
- simple but wasteful
- freshness depends on polling interval

Long polling:

- client waits for the server to respond only when something happens or timeout occurs
- more efficient than regular polling when updates are infrequent
- better freshness without constant short-interval requests

Good rule:

- use regular polling for simple low-value updates
- use long polling when you want fresher updates but still want standard HTTP semantics

8. Long polling vs SSE

Long polling:

- request is reopened after every response
- more universally familiar in older HTTP environments
- can be used where SSE support or infrastructure behavior is uncertain

SSE:

- one long-lived response stream
- simpler for one-way continuous server push in modern browser environments
- usually more efficient than long polling for continuous event streams

Good rule:

- if modern browser-based one-way streaming is acceptable, SSE is usually cleaner
- if you need a simpler fallback pattern over standard request-response behavior, long polling can still be useful

9. Long polling vs WebSockets

Long polling:

- still request-response based
- simpler than full-duplex protocols
- better for moderate server-to-client update patterns

WebSockets:

- persistent full-duplex connection
- better for interactive two-way real-time systems
- more efficient for high-frequency messaging

Good rule:

- choose long polling when full bidirectional communication is unnecessary
- choose WebSockets when both sides need to send messages continuously

10. Timeout design

A long-poll endpoint usually needs a timeout.

Why:

- the server cannot hold requests forever
- proxies or load balancers may close long-running requests
- the client needs a clear reconnect strategy

Common pattern:

- server holds the request for 15 to 30 seconds
- if no event occurs, it returns and the client reconnects immediately

Important keyword: timeout response

- A timeout response is not necessarily an error. It often just means "no new data yet."

11. Scaling considerations

Long polling can become expensive at scale because many clients may keep requests open simultaneously.

Questions to think about:

- How many concurrent waiting requests can the server hold?
- Will the load balancer allow long-held HTTP requests?
- Does each reconnect create heavy authentication or routing cost?
- Where does the waiting event come from in a distributed deployment?

Common architecture:

- clients connect to API servers
- API servers wait for updates from a queue, pub/sub bus, or in-memory event layer
- when an event arrives, the API server completes the held request

12. Reliability considerations

Important questions:

- What happens if the request drops?
- Can the client miss an event between one request ending and the next request starting?
- Should events have IDs so the client can ask for missed updates?

Real example:

- A notification system may return the latest unseen notifications each time so brief disconnect gaps do not permanently lose events.

13. Security

Security concerns include:

- authenticating every long-poll request
- ensuring the user only receives data they are allowed to see
- protecting the endpoint from abuse or reconnect storms

Why this matters:

- because long polling reconnects often, auth and rate limiting behavior need to be thought through carefully

14. Real examples

Use long polling for:

- near-real-time notifications in environments where SSE or WebSockets are not ideal
- job progress in older web stacks
- support or admin dashboards with moderate event frequency
- compatibility-focused systems that must stay on plain HTTP request-response

Do not default to long polling for:

- high-frequency chat or messaging
- multiplayer games
- collaborative editing
- systems with very large numbers of concurrent connected users if SSE or WebSockets are available and more appropriate

15. Staff-level discussion prompts

- How many concurrent waiting clients will there be?
- Is one-way communication enough?
- How expensive is reconnect churn?
- Can the infrastructure tolerate long-held HTTP requests?
- Would SSE be simpler for this browser use case?
- Would WebSockets be more efficient for this event frequency?

Server-Sent Events (SSE) for Senior and Staff-Level Interviews

Server-Sent Events, or SSE, are a browser-friendly way for a server to push a stream of updates to a client over a long-lived HTTP connection. SSE is simpler than WebSockets when communication is one-way from server to client.

At senior and staff level, the important question is when SSE is the right real-time tool and when it is too limited.

1. What SSE is

SSE allows a server to keep an HTTP connection open and continuously send events to the client as new updates happen.

Important keyword: one-way streaming

- One-way streaming means the server sends updates to the client, but the client does not use the same connection to send arbitrary messages back.
- If the client needs to send data, it usually does so through normal HTTP requests.

In browsers, SSE is commonly used through the `EventSource` API.

2. How SSE works

The client makes an HTTP request to an endpoint such as:

- `GET /api/v1/notifications/stream`

The server keeps the response open and periodically sends events formatted as text.

Important keyword: event stream

- An event stream is a sequence of messages sent over one persistent HTTP response.

Common fields in SSE messages:

- `event`: optional event type
- `data`: the payload
- `id`: an event identifier
- `retry`: reconnection hint

3. When to use SSE

SSE is a strong fit when:

- the server needs to push updates to the client
- communication is one-way
- the client is a browser
- you want something simpler than WebSockets
- updates are moderate in frequency rather than ultra-high-frequency

Real examples:

- live notifications in a web app
- progress updates for long-running jobs
- news or sports score updates
- admin dashboards with live status changes
- market or analytics dashboards with moderate real-time refresh

4. Why SSE exists

Without SSE, clients often poll:

- request every 5 seconds
- ask if anything changed

Polling is simpler, but wasteful when updates are infrequent and slow to react when the polling interval is large.

SSE improves this by letting the server push updates immediately over one open connection.

5. Strengths of SSE

- simpler than WebSockets
- uses normal HTTP
- browser-friendly
- good for server-to-browser real-time updates
- easier to operate through HTTP infrastructure than some bidirectional protocols

Why this matters:

- If you only need server push, SSE is often simpler and cheaper than introducing full-duplex communication.

6. Weaknesses of SSE

- one-way only
- text-based, not binary-focused
- not ideal for highly interactive low-latency bidirectional systems
- can be affected by proxy, timeout, or connection-limit behavior
- less appropriate for non-browser clients in some ecosystems compared to gRPC streaming or WebSockets

7. SSE vs polling

Polling:

- the client repeatedly asks for updates
- simpler mental model
- wastes requests when nothing changes

SSE:

- the client opens one connection
- the server pushes updates when they happen
- more efficient for server-to-client live updates

Use SSE over polling when:

- users need fresher updates
- polling traffic would become wasteful
- the communication is mostly one-way

8. SSE vs WebSockets

SSE:

- one-way from server to client
- simpler
- usually better for browser event feeds

WebSockets:

- full-duplex, meaning both sides can send messages over the same connection
- better for chat, games, collaborative editing, and other interactive systems

A good interview answer:

- choose SSE when you only need server push
- choose WebSockets when both sides need to speak continuously

9. Reconnection and event IDs

SSE has built-in reconnection behavior in browsers.

Important keyword: last event ID

- If events include an `id`, the client can reconnect and tell the server the last event it received.
- This helps the server resume or replay missed events, depending on system design.

Why this matters:

- Real-time systems lose connections.
- A robust SSE design should think about reconnection and whether missed events can be replayed.

10. Scaling considerations

SSE uses long-lived connections, so scale depends on how many open connections the servers can handle.

Questions to think about:

- How many concurrent clients will stay connected?
- Will load balancers or proxies time out idle connections?
- Do you need sticky sessions or can any node resume the stream?
- Where do events come from if the app is horizontally scaled?

Common pattern:

- application servers subscribe to a pub/sub layer or message bus
- events are fanned out to connected clients

11. Reliability considerations

Important questions:

- Are updates best-effort or guaranteed delivery?
- Should the client replay missed events after reconnecting?
- What happens if the server restarts?

Real example:

- For live notifications, missing one event may be acceptable if the UI can refetch the latest state.
- For job progress or critical event history, you may need replay support or a backing event log.

12. Security

Security concerns include:

- authenticating the stream request
- ensuring the client only receives events it is allowed to see
- rotating expired auth if the stream stays open a long time

Real example:

- A user should only receive their own notifications, not events for other users or tenants.

13. Real examples

Use SSE for:

- browser notification feeds
- long-running job progress
- live admin dashboards
- moderate-frequency stock or sports updates
- system status pages

Do not default to SSE for:

- chat systems
- multiplayer games
- collaborative editing
- systems that need binary payloads or intense bidirectional messaging

14. Staff-level discussion prompts

- Is the communication one-way or two-way?
- How many concurrent open connections will exist?
- What happens on reconnect?
- Are missed events acceptable?
- Do we need durable replay?
- Would WebSockets or polling be simpler or more appropriate here?

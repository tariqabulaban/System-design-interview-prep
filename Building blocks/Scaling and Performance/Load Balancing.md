Load Balancing for Senior and Staff-Level Interviews

Load balancing is the practice of distributing traffic across multiple servers so the system can scale, stay available, and avoid overloading any single instance. At senior and staff level, you should be able to explain not just what a load balancer is, but where it sits, what kind it is, and how its routing strategy affects availability and correctness.

1. What load balancing is

A load balancer receives traffic and forwards it to one of many backend instances.

Why it exists:

- improve availability
- spread traffic
- support horizontal scaling
- remove unhealthy servers from rotation

2. Where load balancers appear

Common places:

- internet traffic to web servers
- API gateway to backend services
- internal service mesh or east-west traffic

Real example:

- users hit one public endpoint, and the load balancer spreads traffic across many API servers

3. L4 vs L7 load balancing

Layer 4 load balancer:

- routes based on network information like IP and port
- faster and simpler

Layer 7 load balancer:

- routes based on application-level information like HTTP path, host, or headers
- more flexible for modern web and API systems

Why this matters:

- if you need path-based routing like `/api` vs `/admin`, you usually need L7 behavior

4. Common routing strategies

Round robin:

- send requests to backends in rotation

Least connections:

- prefer the backend with the fewest active connections

Weighted routing:

- send more traffic to larger or preferred backends

Hash-based routing:

- choose a backend based on a key like client ID or session ID

Why this matters:

- the right strategy depends on whether requests are stateless, long-lived, or uneven in cost

5. Health checks

Load balancers often run health checks to detect bad instances.

Types:

- TCP checks
- HTTP endpoint checks
- deeper application readiness checks

Why this matters:

- if a backend is unhealthy but still receives traffic, availability drops

6. Sticky sessions

Sticky sessions mean requests from one client are routed to the same backend repeatedly.

Why they exist:

- useful when the backend keeps local session state

Downsides:

- uneven load distribution
- harder failover
- makes scaling and recovery less clean

Senior/staff-level point:

- stateless services are usually better than depending on sticky sessions

7. Load balancing and long-lived connections

Some traffic is not simple request-response.

Examples:

- WebSockets
- SSE
- long polling

Why this matters:

- a backend holding long-lived connections may appear "busy" differently from one serving short HTTP requests
- routing and capacity planning need to account for connection count, not just request rate

8. High availability and failover

Load balancing helps availability by:

- removing failed instances
- routing around unhealthy zones
- spreading traffic across multiple machines or regions

Important question:

- is the load balancer itself highly available?

9. Global vs local load balancing

Local load balancing:

- distributes traffic within one region or cluster

Global load balancing:

- routes users across regions based on geography, latency, or health

Real example:

- send US users to `us-west` when healthy, but fail over to `us-east` if needed

10. Common technologies

Examples:

- Nginx
- HAProxy
- Envoy
- AWS ALB and NLB
- GCP load balancers
- Cloudflare and other edge platforms

11. Common mistakes

- assuming the load balancer automatically solves stateful backend problems
- ignoring health check quality
- forgetting the effect of long-lived connections
- overusing sticky sessions
- not planning for load balancer failure or region failover

12. Real examples

Use load balancing for:

- stateless web tiers
- API servers
- internal service clusters
- WebSocket gateway fleets

13. Staff-level discussion prompts

- Is this traffic stateless or connection-oriented?
- Do we need L4 or L7 routing?
- What routing strategy fits the workload?
- Are sticky sessions required or avoidable?
- How do health checks work?
- What is the failover story if a zone or region dies?

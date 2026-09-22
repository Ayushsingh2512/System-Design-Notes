# Day 02 — Load Balancing

## 1. What is Load Balancing?

A load balancer distributes incoming traffic across multiple backend servers.

Instead of:

Client → Server A

we have:

Client → Load Balancer → Server A / Server B / Server C

The goal is to:
- distribute workload
- prevent one server from becoming overloaded
- remove unhealthy servers from traffic
- enable horizontal scaling
- improve availability

---

## 2. Why Do We Need a Load Balancer?

Suppose one server can handle 10,000 requests/sec but traffic grows beyond that.

Instead of making one machine infinitely powerful:

                    ┌── Server A
Users → Load Balancer ├── Server B
                    └── Server C

We can add more servers.

The load balancer becomes the entry point and distributes requests among them.

---

## 3. Load Balancing Algorithms

### Round Robin

Requests are distributed sequentially:

R1 → A
R2 → B
R3 → C
R4 → A
R5 → B
R6 → C

Works reasonably well when:
- servers have similar capacity
- requests have similar processing cost

Problem:

If:

A = 8 cores
B = 4 cores
C = 2 cores

Simple Round Robin still gives roughly 1/3 of requests to each.

The weaker server may become the bottleneck.

---

### Weighted Round Robin

Servers receive different weights according to their configured capacity.

Example:

A = weight 4
B = weight 2
C = weight 1

Traffic is approximately distributed:

A → 57%
B → 29%
C → 14%

Important:

Weights are configured based on expected capacity. The load balancer isn't necessarily deciding dynamically whether a server is "good enough" for every request.

---

### Least Connections

Send a new connection to the server with the fewest active connections.

Example:

A = 20 connections
B = 5 connections
C = 2 connections

New connection → C

Useful when connection durations vary.

But:

Least connections ≠ least actual work.

Two expensive requests can consume more resources than five cheap requests.

Also, HTTP requests and TCP connections aren't always one-to-one because connections can be reused.

---

## 4. Health Checks

A load balancer should not blindly send traffic to every server.

It can periodically check whether servers are healthy.

             ┌── A ✅
Users → LB ──┼── B ❌
             └── C ✅

The LB removes B from the eligible backend pool.

This means:

Routing algorithms operate on healthy/eligible servers, not necessarily every server.

Health checks help hide individual server failures from users.

---

## 5. Load Balancer Does Not Automatically Make a System Fault Tolerant

A load balancer helps route around failed servers, but it can itself become a problem.

Users → ❌ LB → A/B/C

If there is only one LB and it fails, the whole service can become unreachable.

Therefore production systems can use redundant load balancers or managed load-balancing infrastructure.

---

## 6. L4 vs L7 Load Balancing

### L4 — Transport Layer

Works with information such as:

- IP address
- port
- TCP
- UDP
- connection information

It generally does not need to understand the HTTP request itself.

Example:

Client → L4 LB → Server

Useful when routing based primarily on network/transport information.

---

### L7 — Application Layer

Understands application-level information such as:

- HTTP method
- URL/path
- hostname
- headers
- cookies

Example:

/api/users    → User Service
/api/orders   → Order Service
/api/payments → Payment Service

L7 can make more application-aware routing decisions, but requires more processing and complexity.

---

## 7. L4 and L7 Can Be Combined

They aren't mutually exclusive.

Example:

Clients
   ↓
L4 Load Balancer
   ↓
L7 Load Balancer / Reverse Proxy
   ↓
Services

The architecture depends on requirements.

---

## 8. Reverse Proxy

A reverse proxy sits between clients and backend servers.

Client
  ↓
Reverse Proxy
  ↓
Backend Servers

An L7 load balancer can also act as a reverse proxy.

It can provide:

- request routing
- TLS termination
- health checks
- authentication-related processing
- rate limiting
- traffic management

---

## 9. Sticky Sessions

Suppose a user's session is stored only in Server A's memory.

User → LB → A
           ↓
        Session

If the next request goes to B:

User → LB → B
           ↓
       No session ❌

The server doesn't have the user's local session.

Sticky sessions solve this by keeping the user associated with the same server.

User X → A
User X → A
User X → A

Usually the load balancer uses some form of session affinity/cookie.

### Problems with Sticky Sessions

- uneven traffic distribution
- harder scaling
- server failure can lose local session state
- less flexibility when moving traffic between servers

---

## 10. Better Alternative: Shared Session Store

Instead of keeping sessions inside one server:

A ──┐
B ──┼──→ Redis
C ──┘

Any application server can retrieve the session.

Now:

User → A → Redis
User → C → Redis
User → B → Redis

The session isn't trapped inside one server.

---

## 11. JWT and Stateless Authentication

Another approach is to avoid storing the authentication session on the application server.

The client carries a signed token such as a JWT.

Client
  ↓ JWT
Server A
  ↓
Verify token

The server verifies the token's signature and claims.

The JWT itself contains information such as claims and expiration.

JWT does not automatically "check" the user. The server/application verifies the token.

Redis can still be used alongside JWT for things like:

- token revocation
- refresh-token storage
- rate limiting
- temporary state

---

## 12. Redis Is Not Infinite

Moving state to Redis doesn't eliminate bottlenecks.

Redis itself has limits:

- CPU
- RAM
- network
- storage

It can become a bottleneck.

Possible approaches include:

Replication → redundancy / read scaling
Sharding    → distribute different data across nodes
Clustering  → distribute workload and improve scalability
TTL         → automatically remove temporary data
Eviction    → remove data according to policy when memory is constrained

General principle:

> No component has infinite capacity.

---

## 13. Stateless Application Servers

A scalable application layer generally tries to avoid important request/session state being trapped in one server's local memory.

Example:

             ┌── Server A
Users → LB ──┼── Server B
             └── Server C
                  ↓
             Shared State
            Redis / Database

This makes horizontal scaling and failure recovery easier.

Important:

> Stateless does NOT mean the entire system has no state.

It means the application server doesn't depend on state being stored locally on that particular instance.

---

## 14. Core System Design Insight

A load balancer doesn't magically solve scalability.

It moves us from:

One Server

to:

             ┌── Server A
Users → LB ──┼── Server B
             └── Server C

But now another component can become the bottleneck.

Example:

Users
  ↓
Load Balancer       100k RPS
  ↓
App Servers         100k RPS
  ↓
Redis                80k RPS
  ↓
Database             20k RPS  ← Bottleneck

The bottleneck can move.

This leads to a central system-design mindset:

> Identify the current bottleneck, understand why it exists, and decide how to scale, redesign, or protect that component.

---

## 15. Architecture Is Not One Fixed Pattern

Real systems can combine architectural styles:

- Monolith
- Modular Monolith
- Microservices
- Event-driven architecture
- Serverless
- Layered architecture
- Client-server
- CQRS
- SOA

These aren't mutually exclusive.

For example:

L7 Load Balancer
        ↓
Microservices
        ↓
Redis + PostgreSQL
        ↓
Kafka
        ↓
Workers

The architecture should follow:

Requirements
     ↓
Scale
     ↓
Constraints
     ↓
Required properties
     ↓
Technology choices
     ↓
Trade-offs

Not:

"I know Kafka, so I'll use Kafka."

---

# Key Takeaways

1. Load balancing distributes traffic across multiple backend instances.
2. Round Robin assumes relatively similar capacity/workload.
3. Weighted Round Robin accounts for configured capacity differences.
4. Least Connections considers active connections, but not necessarily actual CPU/work.
5. Health checks prevent traffic from being sent to failed instances.
6. L4 works primarily with transport/network connection information.
7. L7 understands application-level information such as HTTP paths and headers.
8. Sticky sessions keep users attached to particular servers but reduce flexibility.
9. Shared session stores such as Redis allow requests to reach different servers.
10. JWT can provide stateless authentication, but token verification is still performed by the application.
11. Redis can itself become a bottleneck.
12. Stateless application servers make horizontal scaling easier.
13. Adding a load balancer doesn't eliminate bottlenecks; it can simply move the bottleneck elsewhere.
14. Real architectures combine multiple patterns according to requirements and trade-offs.

---

# Mental Model

More Users
    ↓
More Traffic
    ↓
Multiple Servers
    ↓
Load Balancer
    ↓
Traffic Distribution
    ↓
But...
    ↓
New Bottleneck?
    ↓
Cache / DB / Redis / Queue / Network
    ↓
Scale or redesign
    ↓
New bottleneck
    ↓
Repeat

> System design is not about eliminating bottlenecks forever.
> It is about understanding where the bottleneck is, why it exists, what happens when components fail, and making deliberate trade-offs.
# Day 03 — Stateless Architecture

## 1. Stateful vs Stateless

A **stateful application server** keeps important client/request state in its own local memory.

Example:

User → Server A
         ↓
      Session

If the next request goes to Server B:

User → Server B
         ↓
    No session ❌

The problem is that the user's state is tied to a particular server.

This makes horizontal scaling and server replacement harder.

---

## 2. Why Load Balancing Creates the Problem

With multiple servers:

Users
  ↓
Load Balancer
  ↓
A / B / C

A user's requests can reach different servers.

If the session exists only on A:

Request 1 → A → Session available
Request 2 → C → Session unavailable

The issue isn't specifically that Server A is full.

The deeper issue is:

> The user's required state is trapped inside a particular server.

---

## 3. Sticky Sessions

One solution is to keep a user attached to the same server.

User X → A
User X → A
User X → A

This is called **sticky sessions / session affinity**.

### Problems

- uneven traffic distribution
- harder scaling
- server failure can lose local session state
- less flexibility when moving traffic between servers

Sticky sessions solve the routing problem, but don't remove the underlying dependency on local state.

---

## 4. Shared State

Instead of storing session state inside each application server:

A ──┐
B ──┼──→ Redis
C ──┘

Now any application server can retrieve the session.

Request 1 → A → Redis
Request 2 → C → Redis
Request 3 → B → Redis

The application servers become interchangeable.

---

## 5. What Does Stateless Mean?

A **stateless application server** does not depend on important client/request state being stored only in that server's local memory.

Example:

             ┌── Server A
             ├── Server B
Users → LB ──┤
             └── Server C
                  ↓
            Shared State
           Redis / Database

Any server can handle the request.

Important:

> Stateless does NOT mean the entire system has no state.

Redis, PostgreSQL, and object storage can all contain state.

It means the application instance does not depend on owning the state locally.

---

## 6. Why Statelessness Helps Horizontal Scaling

Suppose we have:

Users
  ↓
Load Balancer
  ↓
A / B / C

With stateless application servers:

Request → A
Request → C
Request → B

We can easily add:

A / B / C / D

The new server can immediately start handling requests without needing to migrate important user state from another server.

This makes application instances easier to:

- add
- remove
- restart
- replace
- autoscale

---

## 7. Where Should Different Types of State Live?

Different data has different requirements.

### Application Server Memory

Good for temporary/recreatable information such as:

- connection pools
- compiled templates
- local computation
- temporary caches

If the server dies, the data can simply be recreated.

---

### Redis

Good for fast, shared, temporary or frequently accessed state.

Examples:

- sessions
- cache
- rate-limit counters
- temporary tokens
- distributed locks
- short-lived data

Typical characteristics:

> Very fast + shared + often temporary

Redis can still become a bottleneck and may need replication, clustering, or sharding.

---

### PostgreSQL

Good for durable business/source-of-truth data.

Examples:

- users
- orders
- payments
- invoices
- transactions
- products

Typical requirements:

- durability
- transactions
- consistency
- structured queries
- recovery

Important:

> Redis is not automatically a replacement for a database.

---

## 8. Redis vs PostgreSQL

A useful mental model:

### Redis

> "I need this information very quickly, and it may be temporary or derived."

### PostgreSQL

> "This information is important business data and should be durable."

This is a general rule, not an absolute one. Technologies can be used in different ways depending on requirements.

---

## 9. JWT and Stateless Authentication

JWT can allow authentication information to travel with the request.

Conceptually:

Client
  ↓ JWT
Application Server
  ↓
Verify token

A JWT can contain claims such as:

- user ID
- role
- expiration time

The server verifies the token's signature and claims.

Because any server can independently verify the token:

Request → A
Request → B
Request → C

can all work without a local login session.

---

## 10. Don't Put Everything in a JWT

A JWT should generally contain small pieces of information needed for authentication/authorization.

Avoid putting large or frequently changing data such as:

- shopping cart
- recent orders
- account balance
- entire user profile

inside the token.

Large JWTs increase request size and changing the information generally requires issuing a new token.

---

## 11. Authentication vs Authorization

### Authentication

> Who are you?

Example:

user_id = 123

### Authorization

> What are you allowed to do?

Example:

User:
- can view orders
- cannot issue refunds

Admin:
- can view orders
- can issue refunds

Typical flow:

JWT
 ↓
Identify user
 ↓
Check permissions
 ↓
Allow / Deny

---

## 12. JWT Trade-offs

JWT is not automatically the best solution for every authentication system.

Example:

JWT expires in 1 hour.

User logs out after 2 minutes.

The token may still technically be valid until it expires.

If immediate invalidation is required, the system may need additional mechanisms such as:

- short-lived access tokens
- refresh tokens
- token revocation
- centralized session/revocation state

This can introduce Redis or another centralized component.

Therefore:

> System design is about requirements and trade-offs, not blindly choosing a technology.

---

## 13. Statelessness and Failure

Suppose:

A ✅
B ❌
C ✅

If the application servers are stateless, the load balancer can stop sending traffic to B.

Requests can continue through:

A / C

without losing important state that was trapped inside B.

This is one of the major benefits of stateless application servers.

---

## 14. But Shared State Creates New Dependencies

Suppose:

Application Servers
       ↓
     Redis

If Redis fails, the impact depends on what Redis was responsible for.

### If Redis is only a cache:

The application may be able to retrieve the data from PostgreSQL.

Possible result:

- higher latency
- increased database load
- degraded performance

### If Redis stores user sessions:

Users may lose access to their sessions or authentication flow may fail.

### If Redis contains the only copy of critical business data:

This is a serious architecture problem.

Critical durable business data should generally have an appropriate durable source of truth and recovery strategy.

---

## 15. Moving the Bottleneck

Statelessness does not eliminate bottlenecks.

Before:

Server A
  ↓
Local State

After:

Servers A/B/C
      ↓
    Redis
      ↓
 PostgreSQL

Now Redis or PostgreSQL may become the bottleneck.

General system-design principle:

> Moving state or traffic does not make the underlying problem disappear. It changes where the problem exists.

---

## 16. Architecture Evolution

### Version 1 — Stateful

User
 ↓
Server A
 ↓
Local Memory

Simple, but difficult to scale.

### Version 2 — Multiple Servers

Users
 ↓
Load Balancer
 ↓
A / B / C

Scalable application layer, but local sessions create problems.

### Version 3 — Shared State

Users
 ↓
Load Balancer
 ↓
A / B / C
 ↓
Redis

Application servers no longer need to own session state locally.

### Version 4 — More Complete Architecture

Users
      ↓
Load Balancer
      ↓
A / B / C
      ↓
Redis
      ↓
PostgreSQL

Different components now have different responsibilities.

---

# Key Takeaways

1. Stateful servers keep important state in local memory.
2. Local state becomes a problem when requests can reach different servers.
3. Sticky sessions keep users attached to a server but introduce trade-offs.
4. Shared state allows multiple application servers to access the same information.
5. A stateless application server does not depend on important state being stored locally.
6. Statelessness makes horizontal scaling, autoscaling, replacement, and failure recovery easier.
7. Redis is useful for fast shared/temporary state such as sessions and caches.
8. PostgreSQL is generally used for durable business/source-of-truth data.
9. JWT can allow authentication without storing a login session on each application server.
10. Authentication means identifying the user; authorization means deciding what the user can do.
11. JWT introduces trade-offs such as token revocation and token size.
12. Stateless does not mean the entire system has no state.
13. Shared infrastructure such as Redis can itself become a bottleneck or failure dependency.
14. System design is about understanding requirements, failure modes, bottlenecks, and trade-offs.

---

# Mental Model

Stateful:

User
 ↓
Server A
 ↓
Local State

Problem:
The user depends on Server A.

Stateless:

User
 ↓
Load Balancer
 ↓
A / B / C
 ↓
Shared State
Redis / Database

Benefit:
Any healthy application server can handle the request.

> The goal of stateless architecture is not to eliminate state.
> The goal is to prevent important state from being trapped inside a single application instance.
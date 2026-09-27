# Day 01 — Scalability Fundamentals

## 1. What is Scalability?

Scalability is the ability of a system to handle increasing workload by adding resources while maintaining acceptable performance.

A system should be able to handle growth in:
- Number of users
- Requests
- Data
- Traffic
- Computation

Scalability does NOT mean the system can handle unlimited traffic or every sudden spike automatically.

---

## 2. Load vs Capacity

### Load

The amount of work currently being requested from the system.

Examples:
- 5,000 requests/second
- 100,000 concurrent users
- 2 GB/s of incoming data

### Capacity

The maximum workload a component can handle while meeting its performance requirements.

Example:

Application Server:
- Capacity = 5,000 RPS
- Current Load = 3,000 RPS
- Remaining Capacity = 2,000 RPS

If load exceeds capacity, performance starts degrading or the system may fail.

---

## 3. RPS / QPS

### RPS — Requests Per Second

Example:

10,000 requests / 10 seconds = 1,000 RPS

### QPS — Queries Per Second

Often used when talking specifically about database or query workloads.

Example:

Application → 5,000 RPS
Database → 2,000 QPS

The database can become the bottleneck.

---

## 4. Average Load vs Peak Load

A system should not be designed only around average traffic.

Example:

Normal traffic: 1,000 RPS
Peak traffic: 8,000 RPS

If the system is designed for only 1,000 RPS, it may fail during the peak.

Important concepts:
- Average load
- Peak load
- Traffic spikes
- Capacity headroom

---

## 5. Throughput vs Latency

### Throughput

How much work a system can process in a given amount of time.

Example:

Database throughput = 20,000 queries/sec

### Latency

How long it takes to complete one operation.

Example:

API latency = 100 ms

A system can have:

High throughput + high latency

or

High throughput + low latency

They are different properties.

---

## 6. Vertical Scaling

Vertical scaling means increasing the resources of an existing machine.

Before:

Server
- 4 CPU
- 8 GB RAM

After:

Bigger Server
- 16 CPU
- 64 GB RAM

Advantages:
- Simple
- Often requires fewer architectural changes
- Useful for databases and smaller systems

Limitations:
- Hardware has a maximum size
- Expensive at higher levels
- Can create a single-machine dependency
- Eventually reaches a scaling ceiling

---

## 7. Horizontal Scaling

Horizontal scaling means adding more machines/instances.

Client
   |
Load Balancer
   |
   +---- Server 1
   |
   +---- Server 2
   |
   +---- Server 3

Instead of making one server extremely powerful, we add more servers.

Advantages:
- Can scale much further
- Provides redundancy
- Failure of one instance does not necessarily bring down the service
- Enables distributed architectures

Challenges:
- More complex
- Requires load balancing
- Requires careful handling of shared state
- Distributed systems introduce new failure modes

---

## 8. Load Balancer

A load balancer distributes incoming traffic across multiple servers.

Client
   |
Load Balancer
   |
   +---- Server 1
   |
   +---- Server 2
   |
   +---- Server 3

Its job is not necessarily to distribute requests equally.

It can use strategies such as:
- Round robin
- Weighted routing
- Least connections
- Health checks

A load balancer also helps prevent traffic from being concentrated on unhealthy or overloaded instances.

---

## 9. Bottleneck

A bottleneck is the component that limits the overall capacity of the system.

Example:

Client
   |
App Servers → 10,000 RPS
   |
Database → 2,000 QPS

The database is the bottleneck.

Increasing the number of application servers will not solve the database bottleneck.

Important principle:

> Find the bottleneck before deciding what to scale.

---

## 10. Bottlenecks Can Move

Fixing one bottleneck can expose another.

Before:

App capacity = 10,000 RPS
DB capacity = 2,000 RPS

DB = bottleneck

After improving the DB:

App capacity = 10,000 RPS
DB capacity = 20,000 RPS

Now the application layer may become the bottleneck.

Therefore system design is iterative:

Measure
   ↓
Find bottleneck
   ↓
Improve it
   ↓
Measure again
   ↓
Find next bottleneck

---

## 11. Core System Design Mindset

Do not start with:

"Which technology should I use?"

Start with:

Requirements
   ↓
Estimate workload
   ↓
Understand capacity
   ↓
Find bottlenecks
   ↓
Choose architecture
   ↓
Scale the bottleneck
   ↓
Handle failures
   ↓
Evaluate trade-offs

Technology choices should follow the requirements.

---

## 12. Example

Suppose an application receives:

10,000 RPS

Architecture:

             +---- App Server 1
             |
Client → LB -+---- App Server 2
             |
             +---- App Server 3
                       |
                    Database

Suppose:

Each App Server = 4,000 RPS
Database = 5,000 QPS

Application capacity:

3 × 4,000 = 12,000 RPS

So the application layer can theoretically handle the traffic.

But the database can handle only:

5,000 QPS

Therefore:

Database = bottleneck

Adding more application servers alone will not solve the problem.

---

## 13. Key Takeaways

- Scalability = ability to handle increasing workload by adding resources while maintaining acceptable performance.
- Load = work being requested.
- Capacity = work a component can handle.
- RPS/QPS measure request/query rate.
- Throughput and latency are different.
- Vertical scaling = make a machine bigger.
- Horizontal scaling = add more machines.
- Load balancers distribute traffic across instances.
- Bottlenecks limit system capacity.
- Fixing one bottleneck can expose another.
- Architecture should be driven by requirements and constraints.
- Always ask: "What is the current bottleneck?"

---

## Questions I Should Be Able to Answer

1. What is scalability?
2. What is the difference between load and capacity?
3. What is RPS?
4. What is the difference between throughput and latency?
5. What is the difference between vertical and horizontal scaling?
6. Why can't we vertically scale forever?
7. Why do we need a load balancer?
8. What is a bottleneck?
9. Why doesn't adding more app servers always solve performance problems?
10. Why can bottlenecks move after scaling?
11. Why should requirements come before technology choices?

---

## Learning Principle

> Learn → Understand → Apply → Document

These notes are a summary of my understanding, not a replacement for learning the concepts deeply.
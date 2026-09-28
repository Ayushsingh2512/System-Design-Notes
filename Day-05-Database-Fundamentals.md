# Day 05 — Database Fundamentals

## 1. Why Do We Need a Database?

A database provides a persistent place to store application data.

For example:

- users
- products
- orders
- payments
- messages
- transactions

Unlike application memory, database data should survive when an application server restarts.

Basic architecture:

Users
  ↓
Application
  ↓
Database

---

## 2. Database as the Source of Truth

In many systems, the database acts as the durable source of truth.

Example:

PostgreSQL
    ↓
Users
Orders
Payments
Products

A cache may contain copies of frequently accessed data, but the database generally holds the durable business data.

This gives us:

Application
    ↓
Cache → Fast access
    ↓
Database → Durable source of truth

---

## 3. SQL vs NoSQL

Two broad database categories are:

### SQL / Relational Databases

Examples:

- PostgreSQL
- MySQL
- MariaDB

Data is organized into tables with defined relationships.

Example:

Users

| id | name  | email |
|----|-------|-------|
| 1  | Ayush | ...   |
| 2  | Rahul | ...   |

Orders

| id | user_id | amount |
|----|---------|--------|
| 1  | 1       | 500    |
| 2  | 2       | 800    |

The `user_id` can connect an order to a user.

---

### NoSQL Databases

NoSQL is a broad category that includes different database models.

Examples:

- document databases
- key-value stores
- wide-column databases
- graph databases

Examples of technologies:

- MongoDB
- DynamoDB
- Cassandra
- Redis

NoSQL databases can be useful when the data model, scale, access patterns, or availability requirements don't fit a traditional relational approach well.

---

## 4. Why Relationships Matter

Suppose we have:

User → Orders

One user can have many orders.

In a relational database:

Users
  ↓
user_id
  ↓
Orders

Example:

User:

id = 123
name = Ayush

Orders:

order_id = 1
user_id = 123
amount = 500

order_id = 2
user_id = 123
amount = 900

The relationship is explicitly represented.

Relational databases are particularly useful when applications have structured data and relationships that need to be queried consistently.

---

## 5. Primary Key

A primary key uniquely identifies a row.

Example:

Users

id | name
---|------
1  | Ayush
2  | Rahul

Here:

id

is the primary key.

A primary key should uniquely identify each record.

---

## 6. Foreign Key

A foreign key creates a relationship between tables.

Example:

Users:

id | name
---|------
1  | Ayush

Orders:

id | user_id | amount
---|---------|-------
10 | 1       | 500

`Orders.user_id` references `Users.id`.

This allows the database to represent relationships between entities.

---

## 7. Database Queries

Applications interact with databases using queries.

Example:

SELECT * FROM users WHERE id = 123;

The database receives the query and determines how to retrieve the required data.

For example:

Application
    ↓
SQL Query
    ↓
PostgreSQL
    ↓
Result

At scale, the efficiency of these queries becomes extremely important.

---

## 8. Indexes

Suppose a users table contains:

10 rows

A simple scan is cheap.

But imagine:

1 billion rows.

If we search:

WHERE email = 'example@gmail.com'

and the database checks every row:

Row 1
Row 2
Row 3
...
Row 1,000,000,000

the query can become very expensive.

An index can make lookups much faster.

Conceptually:

Without index:

Query → Scan many rows → Find result

With index:

Query → Index → Relevant row

---

## 9. Index Trade-off

Indexes are not free.

Benefits:

- faster reads
- faster lookups
- faster filtering/sorting for supported queries

Costs:

- additional storage
- slower writes
- index maintenance
- additional memory/I/O

Every insert, update, or delete may require the relevant indexes to be updated.

Therefore:

> Don't create indexes on every column blindly.

Indexes should be created based on actual query patterns.

---

## 10. Transactions

A transaction groups multiple database operations into one logical unit of work.

Example: transferring ₹500.

We need:

1. subtract ₹500 from Account A
2. add ₹500 to Account B

We don't want:

Account A → ₹500 deducted
Account B → money not added

A transaction helps ensure the operations are handled as one unit according to the database's transaction guarantees.

Conceptually:

BEGIN TRANSACTION

Subtract ₹500 from A
Add ₹500 to B

COMMIT

If something goes wrong:

ROLLBACK

---

## 11. ACID

Relational databases commonly provide transaction guarantees described by ACID.

### A — Atomicity

A transaction happens completely or not at all.

Example:

Either both sides of a transfer succeed, or neither does.

---

### C — Consistency

A successful transaction should leave the database satisfying its defined rules and constraints.

Example:

A foreign key constraint should not be violated after a successful transaction.

---

### I — Isolation

Concurrent transactions should behave according to the database's isolation guarantees.

One transaction should not incorrectly interfere with another.

---

### D — Durability

Once a transaction is committed, its data should survive appropriate failures such as a database process restart.

---

## 12. Why Transactions Matter in System Design

Transactions become important when multiple operations must maintain a business invariant.

Examples:

- payments
- bank transfers
- inventory
- order creation
- ticket booking
- budget reservation

Example:

If an AI Cost Guardrail has:

budget = ₹1000

and two requests arrive simultaneously:

Request A → reserve ₹700
Request B → reserve ₹700

Without proper concurrency control, both might believe enough budget exists.

The system could accidentally reserve:

₹700 + ₹700 = ₹1400

even though the budget is only ₹1000.

Database transactions and concurrency control can help prevent this.

---

## 13. Concurrency

Concurrency means multiple operations are happening at overlapping times.

Example:

Two users purchase the last available product simultaneously.

Initial stock:

stock = 1

Request A → buy product
Request B → buy product

Both requests may read:

stock = 1

If the system handles this incorrectly, both could succeed.

The database needs mechanisms to safely handle concurrent operations.

This becomes increasingly important at scale.

---

## 14. Connection Between Database and System Design

A database isn't simply:

"Store data here."

We need to ask:

- How much data?
- How many reads?
- How many writes?
- What latency is required?
- What consistency is required?
- What happens if the database fails?
- Can it scale vertically?
- Can it scale horizontally?
- Do we need replication?
- Do we need partitioning?
- What queries are most common?

These questions determine the database architecture.

---

## 15. Vertical Scaling of a Database

One approach is to make the database machine more powerful.

Example:

Before:

8 CPU cores
32 GB RAM

After:

32 CPU cores
128 GB RAM

Advantages:

- relatively simple
- fewer distributed-system complications

Limitations:

- hardware limits
- increasing cost
- eventually reaches a ceiling
- still creates dependence on a database instance

---

## 16. Horizontal Database Scaling

Instead of one database machine:

Database A
Database B
Database C

Different techniques can be used depending on the workload.

Examples:

- replication
- read replicas
- partitioning
- sharding

These solve different problems and should not be treated as interchangeable terms.

We will study them in the following days.

---

## 17. The Database Can Become the Bottleneck

Consider:

Users
  ↓
Load Balancer
  ↓
10 Application Servers
  ↓
PostgreSQL

The application layer may scale horizontally.

But if PostgreSQL can only handle the required workload up to a certain point:

10 App Servers
      ↓
PostgreSQL ← Bottleneck

Adding more application servers won't necessarily solve the problem.

This connects directly to our earlier system-design principle:

> The bottleneck can move.

---

# Key Takeaways

1. A database provides persistent storage for application data.
2. Databases often act as the durable source of truth.
3. SQL databases organize structured data into related tables.
4. NoSQL is a broad category containing several different database models.
5. Primary keys uniquely identify records.
6. Foreign keys represent relationships between tables.
7. Indexes can make reads significantly faster but add storage and write overhead.
8. Transactions group operations into a logical unit of work.
9. ACID describes important transaction guarantees.
10. Concurrency creates problems when multiple operations modify the same data simultaneously.
11. Databases can become bottlenecks even when application servers scale horizontally.
12. Database scaling can involve vertical scaling, replication, partitioning, and sharding.
13. Database architecture should be driven by workload, consistency, latency, availability, and scale requirements.

---

# Mental Model

Application
    ↓
Database

As the system grows:

More Users
    ↓
More Requests
    ↓
More Database Reads/Writes
    ↓
Database Bottleneck
    ↓
Optimize Queries / Indexes
    ↓
Caching
    ↓
Vertical Scaling
    ↓
Replication
    ↓
Partitioning / Sharding

The important idea:

> A database is not just a storage box. At scale, its access patterns, consistency requirements, transactions, concurrency, and scaling strategy become major parts of system design.
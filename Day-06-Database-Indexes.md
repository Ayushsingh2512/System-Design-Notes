# Day 06 — Database Indexes

## 1. The Problem: Finding Data

Suppose a table contains 1 billion users.

We run:

SELECT * FROM users WHERE email = 'ayush@example.com';

Without an appropriate index, the database may need to examine a large number of rows.

Conceptually:

Query
  ↓
Scan rows
  ↓
Find matching row

As the table grows, this can become expensive.

---

## 2. What Is an Index?

An index is an additional data structure maintained by the database to make certain queries faster.

Conceptually:

Without index:

Query
  ↓
Many rows
  ↓
Find result

With index:

Query
  ↓
Index
  ↓
Relevant row(s)

A useful analogy is a book.

Without an index:

"Where is the topic about databases?"

You may need to scan page after page.

With an index:

Database → Page 250

You can jump much closer to the required information.

---

## 3. Example

Suppose we have:

users

id | name  | email
---|-------|----------------
1  | Ayush | ayush@gmail.com
2  | Rahul | rahul@gmail.com
3  | Aman  | aman@gmail.com

Query:

SELECT * FROM users WHERE email = 'rahul@gmail.com';

If email is indexed, the database can use the index to locate the matching row efficiently.

---

## 4. Indexes Are Not Free

Indexes improve certain reads, but they introduce costs.

Benefits:

- faster lookups
- faster filtering
- faster sorting for supported queries
- reduced amount of data that must be scanned

Costs:

- additional storage
- additional memory usage
- slower INSERT operations
- slower UPDATE operations
- slower DELETE operations
- index maintenance

Why do writes become slower?

When a row changes, the database may also need to update the relevant indexes.

Therefore:

> An index is a trade-off between read performance and write/storage overhead.

---

## 5. Don't Index Every Column

It may seem logical to create an index on every column.

But this can be harmful.

Suppose a table has:

id
name
email
age
city
phone
created_at

Creating many indexes means:

- more storage
- more maintenance
- slower writes
- more memory pressure

Indexes should be based on actual query patterns.

Ask:

> Which queries are frequent or expensive?

Then design indexes around those queries.

---

## 6. Indexing Based on Query Patterns

Suppose the application frequently runs:

SELECT * FROM users WHERE email = ?;

An index on:

email

may be useful.

If the application frequently runs:

SELECT * FROM orders WHERE user_id = ?;

An index on:

user_id

may be useful.

The correct index depends on how the application accesses the data.

---

## 7. Primary Keys and Indexes

A primary key is commonly backed by an index in relational databases.

Example:

users

id | name
---|------
1  | Ayush
2  | Rahul
3  | Aman

Query:

SELECT * FROM users WHERE id = 2;

The database can efficiently locate the row using the primary-key index.

---

## 8. Unique Indexes

A unique index can enforce uniqueness while also supporting efficient lookups.

Example:

CREATE UNIQUE INDEX idx_users_email
ON users(email);

Now the database can enforce:

No two users can have the same email.

At the same time, queries searching by email can use the index.

---

## 9. Composite Indexes

Sometimes queries filter using multiple columns.

Example:

SELECT *
FROM orders
WHERE user_id = 123
AND status = 'pending';

A composite index can contain multiple columns.

For example:

(user_id, status)

This can be useful when the application frequently queries using those columns together.

---

## 10. Order of Columns Matters

Consider:

(user_id, status)

This is not necessarily equivalent to:

(status, user_id)

The order of columns in a composite index affects which queries can efficiently use it.

A common mental model is that the index is organized first by the first column, then by the next column within that ordering.

Therefore:

> Composite index design should follow actual query patterns.

---

## 11. Selectivity

Selectivity describes how well a column distinguishes between rows.

Example:

Suppose we have 1 million users.

Column:

gender

might have only a few possible values.

Column:

email

is much more unique.

An index on a highly selective column can often be more useful for equality lookups than an index on a column with very few distinct values.

But the usefulness of an index depends on the query, database engine, data distribution, and execution plan.

---

## 12. Index Does Not Guarantee a Faster Query

Having an index doesn't mean the database must use it.

The database query planner considers factors such as:

- table size
- number of matching rows
- available indexes
- estimated cost
- statistics
- query structure

Sometimes a full table scan can actually be cheaper than using an index.

Example:

If a query returns a very large percentage of the table, using the index may not provide much benefit.

Therefore:

> Indexes provide an option to the query planner, not a guarantee that every query will use them.

---

## 13. Query Planner

Modern databases have query planners/optimizers.

Conceptually:

SQL Query
   ↓
Query Planner
   ↓
Possible execution strategies
   ↓
Choose estimated low-cost plan
   ↓
Execute

For example, the planner may choose:

- index scan
- sequential/full table scan
- different join strategies
- different ordering of operations

Understanding query plans becomes extremely important when optimizing databases.

---

## 14. EXPLAIN

Databases provide tools to inspect how a query is executed.

For PostgreSQL:

EXPLAIN SELECT * FROM users WHERE email = 'ayush@example.com';

This can show information about the query plan.

For example, you may discover:

Sequential Scan

when you expected:

Index Scan

This helps diagnose slow queries.

---

## 15. Index and Write-Heavy Systems

Suppose an application performs:

10,000 reads/sec
100 writes/sec

Indexes can be very valuable because the workload is heavily read-oriented.

But imagine:

100 reads/sec
10,000 writes/sec

A large number of indexes may create significant write overhead.

Therefore:

> Index design depends on the read/write workload.

---

## 16. Indexes and Memory

Indexes consume storage and may also benefit from being cached in memory.

A very large index can create memory and I/O pressure.

At large scale, database performance depends not only on CPU but also on:

- RAM
- disk I/O
- cache behavior
- network
- query patterns
- index size

---

## 17. Indexes and Caching

Indexes and application-level caching solve different problems.

Caching:

Application
    ↓
Redis
    ↓
Fast cached response

Indexing:

Application
    ↓
Database
    ↓
Index
    ↓
Find required rows efficiently

They can work together.

Example:

User requests a popular product.

Cache hit:
Application → Redis → Product

Cache miss:
Application → PostgreSQL → Index → Product

The index helps the database efficiently retrieve the data when the cache doesn't have it.

---

## 18. Indexes Can Become a Bottleneck

At large scale, an index can itself become expensive.

For example:

- very large index
- frequent writes
- frequent index updates
- memory pressure
- storage/I/O pressure

Therefore:

> Database optimization is not simply "add an index."

We need to understand the workload and measure the actual behavior.

---

## 19. The System Design Connection

Consider:

Users
  ↓
Load Balancer
  ↓
Application Servers
  ↓
Redis
  ↓
PostgreSQL

Suppose PostgreSQL becomes slow.

Before adding another database server, investigate:

1. Are queries inefficient?
2. Are the correct indexes present?
3. Are queries scanning too many rows?
4. Is the database CPU-bound?
5. Is it memory-bound?
6. Is disk I/O the problem?
7. Is the workload read-heavy or write-heavy?

Only after understanding the bottleneck should we decide what scaling technique is appropriate.

---

## 20. Optimization Before Scaling

A useful progression is:

Slow Query
   ↓
Inspect Query
   ↓
Check Execution Plan
   ↓
Add/modify appropriate Index
   ↓
Measure Again
   ↓
Still a bottleneck?
   ↓
Caching / Read Replicas / Partitioning / Sharding
   ↓
Choose based on requirements

This prevents us from using distributed-system complexity to solve a problem that could have been fixed with a better query or index.

---

# Key Takeaways

1. An index is a data structure that helps a database find data efficiently.
2. Indexes can dramatically improve certain read queries.
3. Indexes require additional storage and maintenance.
4. Indexes can slow down INSERT, UPDATE, and DELETE operations.
5. Don't create indexes on every column.
6. Index design should follow actual query patterns.
7. Primary keys are commonly backed by indexes.
8. Unique indexes can enforce uniqueness and support lookups.
9. Composite indexes contain multiple columns.
10. Column order matters in composite indexes.
11. Selectivity affects the usefulness of an index.
12. The query planner decides whether an index is actually useful for a particular query.
13. EXPLAIN can help understand how a query is executed.
14. Index strategy depends heavily on the read/write workload.
15. Indexes and caching solve different problems and can work together.
16. Database optimization should begin by identifying the actual bottleneck.
17. Adding more servers is not always the first or correct solution.

---

# Mental Model

Without a useful index:

Query
  ↓
Large amount of data
  ↓
Expensive scan
  ↓
Result

With an appropriate index:

Query
  ↓
Index
  ↓
Relevant rows
  ↓
Result

System-design progression:

Slow Database
     ↓
Measure
     ↓
Inspect Query
     ↓
Inspect Execution Plan
     ↓
Optimize Index/Query
     ↓
Measure Again
     ↓
Still insufficient?
     ↓
Caching / Replication / Partitioning / Sharding

Core idea:

> Indexes trade additional storage and write/maintenance cost for faster access to data.
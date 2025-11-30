
---

## 1. Relational vs Non-Relational

### Q1. What is the difference between a relational database and a non-relational (NoSQL) database?

```text
A relational database stores data in structured tables with rows and columns, and uses relationships (foreign keys) between tables. It has a fixed schema and supports SQL and ACID transactions.

A non-relational (NoSQL) database stores data in more flexible formats such as documents, key–value pairs, wide columns, or graphs. The schema is more flexible, and it’s often optimised for scalability and specific access patterns rather than strict relational constraints.
````

---

### Q2. When would you choose a relational database like PostgreSQL over a NoSQL database like MongoDB?

```text
I’d choose a relational database when the data is well-structured and relationships are important, for example users, orders, payments.

It’s a good choice when:
- I need strong consistency and transactions (e.g. money moving between accounts).
- I want to enforce constraints like uniqueness and foreign keys.
- I need to run complex queries and joins, e.g. for reporting or analytics.

```

---

### Q3. Give an example of a use case where a NoSQL database might be a better fit than a relational database. Explain why.

```text
A large-scale social feed or microblogging platform can be a good fit: write-heavy, huge volume of events, and quite simple access patterns (e.g. “get latest posts for user X”).

A document or key–value store can:
- Scale horizontally more easily.
- Store flexible payloads as the product evolves.
- Optimise for high write throughput and fast simple reads rather than complex joins.
```

---

## 2. Schema Design & Relationships

### Q4. How would you model users, posts, and comments in a relational database?

Describe the tables and the relationships between them.

```text
I’d have three tables:

- users: id (PK), email, password_hash, created_at, ...
- posts: id (PK), user_id (FK to users.id), title, body, created_at, ...
- comments: id (PK), post_id (FK to posts.id), user_id (FK to users.id), body, created_at, ...

Relationships:
- One user → many posts.
- One post → many comments.
- One user → many comments.
Foreign keys enforce that posts always point to an existing user, and comments point to existing posts and users.
```

---

### Q5. Explain one-to-many and many-to-many relationships, using examples from a typical web application.

```text
One-to-many:
- One user can have many posts, but each post belongs to exactly one user.
- Implemented with posts.user_id as a foreign key to users.id.

Many-to-many:
- Many users can like many posts.
- Implemented with a join table likes(user_id, post_id) where both columns are foreign keys.
```

---

### Q6. How do you enforce data integrity between related tables?

For example, between `users` and `posts`.

```text
By using foreign keys and constraints.

For example, posts.user_id is a foreign key referencing users.id. That ensures:
- You cannot insert a post with a non-existent user_id.
- If a user is deleted (depending on ON DELETE behaviour), their posts are either deleted or rejected, according to the chosen constraint.

I also use NOT NULL, UNIQUE, and appropriate data types to keep data consistent.

```

---

## 3. Queries & Basic Performance

### Q7. Suppose you have `users` and `posts`.

How would you conceptually fetch “the latest 20 published posts with the author’s name”?

(You don’t have to write exact SQL, just describe the idea.)

```text
Conceptually:
- Select from posts where published = true.
- Join posts with users on posts.user_id = users.id to get the author info.
- Order by posts.created_at descending.
- Limit the result to 20 rows.

In plain words: “get the 20 most recent published posts, joined to the user table to include the author’s name.”
```

---

### Q8. How does pagination with `LIMIT` and `OFFSET` work, and what are its drawbacks at large scale?

```text
LIMIT defines how many rows to return, OFFSET defines how many rows to skip.

For example, LIMIT 20 OFFSET 4000 means:
- The database scans and discards the first 4000 matching rows.
- Then returns the next 20.

Drawbacks at large scale:
- Large offsets get slower because the DB still has to skip many rows.
- Users can see inconsistent pages if new rows are inserted or deleted while they’re paging.
```

---

### Q9. What is keyset pagination, and when would you consider using it instead of `LIMIT`/`OFFSET`?

```text
Keyset pagination (or cursor-based pagination) uses a stable sort key (like id or created_at) instead of OFFSET.

Example idea:
- “Give me 20 posts where created_at < last_seen_created_at, ordered by created_at DESC.”

It scales better for large datasets because the DB can seek directly to a position using an index, instead of counting and skipping thousands of rows. I’d consider it when pages go deep or when performance with OFFSET starts to degrade.
```

---

## 4. Indexes & Query Optimisation

### Q10. What is an index in a database, and why is it useful?

```text
An index is an additional data structure (often a B-tree) that the database maintains to speed up lookups on one or more columns.

Instead of scanning every row in a table, the DB can use the index to quickly find rows that match a condition (e.g. WHERE email = ... or WHERE post_id = ...).
```

---

### Q11. What are the trade-offs of adding indexes to a table?

```text
Benefits:
- Faster reads for queries that use the indexed columns (WHERE, JOIN, ORDER BY, GROUP BY).

Costs:
- Slower writes: every INSERT/UPDATE/DELETE has to update the index as well.
- Extra storage.
- If you add too many indexes or index the wrong columns, you can hurt overall performance rather than help it.
```

---

### Q12. Imagine a `comments` table with columns: `id`, `post_id`, `user_id`, `body`, `created_at`.

Which columns would you index and why?

```text
Common choices:
- Index on post_id: to quickly fetch all comments for a given post.
- Possibly a composite index on (post_id, created_at): to get comments for a post ordered by time efficiently.
- Optionally an index on user_id if I frequently fetch “all comments by a given user”.

I wouldn’t index body, because it’s text and unlikely to be used as a main filter in queries.
```

---

### Q13. How would you approach a situation where a query is suddenly slow in production?

(High-level steps, not specific commands.)

```text
High-level approach:
1. Confirm the scope: which endpoint is slow, under what load, and since when.
2. Look at the slow query log / monitoring to identify the exact SQL.
3. Inspect the query plan to see if the DB is doing a full table scan or using the wrong index.
4. Check for obvious issues:
   - Missing or suboptimal indexes.
   - Unnecessary joins.
   - Selecting more data than needed.
5. Optimise:
   - Add or adjust indexes.
   - Rewrite the query if needed.
   - Consider caching results for expensive, read-heavy queries.
6. Re-test under realistic load and monitor impact.
```

---

## 5. Transactions & Consistency

### Q14. What is a transaction, in the context of a database?

```text
A transaction is a group of database operations that are treated as a single unit of work: they either all succeed (commit) or all fail (rollback).

It helps keep the data consistent when multiple related changes need to happen together.
```

---

### Q15. Explain the ACID properties of a transaction in simple, intuitive terms.

```text
ACID:

- Atomicity:
  All operations in the transaction succeed or none of them do. No half-written state.

- Consistency:
  The database moves from one valid state to another; constraints like foreign keys and uniqueness are respected.

- Isolation:
  Concurrent transactions don’t interfere in a way that breaks the guarantees. Each transaction behaves as if it’s alone, up to the chosen isolation level.

- Durability:
  Once a transaction is committed, the changes are persisted. A crash won’t silently undo committed data.
```

---

### Q16. Give an example where multiple operations MUST be wrapped in a single transaction.

```text
Example: when a user transfers money or credits between two accounts.

Operations:
- Decrease the balance in account A.
- Increase the balance in account B.
- Insert a record into a transfers table.

These must either all succeed together or all fail; otherwise you could lose or create money. So they must be wrapped in one transaction.
```

---

### Q17. What is an isolation level, and why doesn’t every system run at the strongest isolation level?

```text
An isolation level defines how visible other transactions’ intermediate changes are during your transaction (how “isolated” it is from concurrent work).

Higher isolation (like serializable) reduces anomalies (dirty reads, non-repeatable reads, phantoms) but increases locking and overhead, which can reduce throughput.

Not every system runs at the strongest isolation because it can hurt performance and concurrency. Often the default (like Read Committed) is a good balance between correctness and performance for typical web applications.
```

---

## 6. Caching & Redis

### Q18. Why would you introduce Redis into a backend system that already has a relational database?

```text
To offload repeated read-heavy work from the main database and reduce latency.

Redis is an in-memory store, so it’s much faster for:
- Frequently accessed data that doesn’t change constantly.
- Session data, rate-limiting counters, or small config values.

Using Redis as a cache can:
- Reduce database load.
- Improve response times.
- Smooth out traffic spikes.
```

---

### Q19. Explain the “cache-aside” (lazy loading) pattern in your own words.

```text
Cache-aside means:
- On read: first check the cache.
  - If the data is there (cache hit), return it.
  - If not (cache miss), read from the database, put the result into the cache, then return it.
- On write: update the database and either invalidate or update the relevant cache entry.

The application is responsible for moving data in and out of the cache “on the side” of normal DB operations.
```

---

### Q20. What kinds of data would you typically cache in a web application, and what would you not cache?

```text
Typically cache:
- Data that is read frequently and doesn’t change constantly:
  - “Top posts” lists, product catalogues, configuration, precomputed aggregates.
- User sessions or auth tokens.
- Results of expensive queries.

Usually don’t cache:
- Highly sensitive data (unless carefully managed).
- Data that changes on almost every request (the overhead of caching may not help).
- Very large blobs where caching doesn’t give much benefit.
```

---

### Q21. What is a TTL (Time To Live) in caching, and how would you choose a good TTL value?

```text
TTL is how long a cache entry remains valid before it automatically expires.

Choosing a TTL is a trade-off between freshness and load:
- If data must be very fresh (e.g. stock levels), TTL should be short or invalidation should be explicit.
- For things like “top posts” or “trending items”, a TTL of seconds to minutes is often fine.

I’d start with a reasonable guess (e.g. 30–60 seconds for a trending list), monitor behaviour, and adjust based on how stale data is allowed to be and how much load I want to take off the DB.
```

---

### Q22. How can inconsistencies arise between the cache and the database?

Give a simple example and how you would mitigate it.

```text
Example:
- A post is updated in the database (title changed).
- But the old version is still stored in Redis and served from the cache.
- Users keep seeing stale data until the cache expires.

Mitigations:
- On write, explicitly delete or update the relevant cache entries (write-through or explicit invalidation).
- Use a reasonable TTL so stale data eventually disappears even if invalidation is missed.
- In critical cases, design the system so the source of truth (DB) is still consulted when needed.
```

---

### Q23. How would you use Redis (or a similar store) for rate limiting in an API?

```text
Basic idea:
- Use a Redis key per user or per API token, e.g. "rate:user:123".
- Every time a request comes in, increment the counter and set an expiry (TTL) for the time window (e.g. 1 minute).
- If the counter exceeds the allowed limit within that window, reject further requests with 429 Too Many Requests.

Redis is good for this due to fast increments and atomic operations.
```

---

## 7. Short Scenario Questions

### Q24. You have an endpoint `/top-posts` that is read very frequently and runs an expensive aggregation query.

How would you combine the database and a cache to make this fast and scalable?

```text
I’d keep the aggregation logic in the database but cache the result in Redis.

Flow:
- When /top-posts is called, first check Redis for a cached JSON list of top posts.
- If it exists and is not expired, return it directly from Redis.
- If not, compute it from the DB (run the aggregation query), store the result in Redis with a TTL (e.g. 30–60 seconds), and return it.

This way:
- The expensive query is run infrequently.
- Most requests are served from the cache with low latency.
```

---

### Q25. A client wants to migrate their monolithic on-prem application to the cloud.

From a database and caching perspective, what are the main things you would pay attention to?

```text
Key concerns:

- Database:
  - How to migrate data safely (backups, migration plan, cutover strategy).
  - Whether to use a managed cloud database (e.g. managed Postgres).
  - Network latency between app and DB, and how that impacts performance.
  - Security: encryption, access control, backups, disaster recovery.

- Caching:
  - Whether to introduce a cache layer (e.g. managed Redis) to reduce DB load.
  - Identify read-heavy endpoints or queries that would benefit from caching.
  - Decide cache keys, TTLs, and invalidation strategies.

- Observability:
  - Set up monitoring for DB load, query performance, and cache hit rate so we can tune the setup after migration.
```

---

### Q26. You notice increased load on your database after launching a new feature.

List a few database- and cache-related checks or changes you would consider.

```text
I’d consider:

- Database side:
  - Check which queries or endpoints increased in frequency.
  - Look at slow queries for the new feature and see if they need indexes.
  - Verify that queries only fetch necessary columns and rows.

- Caching:
  - Identify whether we can cache some of the new read-heavy queries.
  - Check if existing caches still have a good hit rate or if patterns changed.
  - Introduce Redis caching for expensive or frequently used reads.

- Design:
  - If writes increased significantly, consider simplifying write paths or batching where appropriate.
  - For very hot data, consider denormalising or precomputing aggregates that are expensive to compute on the fly.
```

---


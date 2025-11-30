
---

## 1. High-Level System Design Basics

### Q1. When someone says “design a system”, what are the main aspects you think about?

```text
I think about:
- Requirements: what the system should do and for whom.
- Constraints: traffic, latency, data size, regulatory, deadlines.
- Architecture: main components and how they communicate (APIs, queues, DBs).
- Data: how it’s stored, accessed, and kept consistent.
- Non-functionals: scalability, reliability, security, observability, cost.
Then I pick technologies that fit these needs rather than starting from tools.
```

---

### Q2. In a typical web application, what are the main components between a user’s browser and the database?

```text
Conceptually:
browser → DNS → CDN (optional) → load balancer / API gateway → backend application servers → cache (e.g. Redis) → database.

Not every system has all of these, but that’s the typical path between user and DB.

```

---

### Q3. How do you usually start a system design discussion in an interview?

```text
I start by clarifying:
- Use cases: what the system must do.
- Users: who uses it and how.
- Scale: rough traffic (requests/sec), data size, growth.
- Non-functional needs: latency, availability, consistency requirements.

Then I restate the problem in my own words and propose a simple high-level design before going into details.
```

---

## 2. Monoliths, Microservices & Boundaries

### Q4. What is a monolithic architecture? What are its pros and cons?

```text
A monolith is a single deployable application that contains most or all of the system’s functionality.

Pros:
- Simple to develop, test, and deploy (one codebase, one artifact).
- Easy to start with and great for small teams.
- No distributed-system complexity between services.

Cons:
- Becomes harder to change and deploy independently as it grows.
- Scaling is coarse-grained (you scale everything, not just hot parts).
- Tighter coupling between modules; one bug can affect the whole app.

```

---

### Q5. What is a microservices architecture? What are its pros and cons?

```text
Microservices architecture splits the system into multiple smaller services, each owning a specific business capability and data.

Pros:
- Independent deployment and scaling per service.
- Teams can work more independently.
- Technology choices can differ per service when needed.

Cons:
- Higher operational complexity (networking, observability, deployments).
- Distributed-system problems: failures, timeouts, data consistency.
- More complex debugging and testing across service boundaries.

```

---

### Q6. When might you recommend **keeping** or **starting** with a monolith instead of microservices?

```text
I’d recommend a monolith when:
- The product is early-stage and requirements are still changing.
- The team is small and operational overhead must be low.
- Scale and complexity don’t yet justify microservices.

You can still modularise the code internally and later extract services if the system and team grow enough to need them.

```

---

### Q7. How would you think about splitting a system into services? What is a good “service boundary”?

```text
I’d split around business capabilities or domains, not just technical layers.

A good service boundary:
- Aligns with a clear business concept (e.g. Users, Billing, Notifications).
- Has high internal cohesion and low coupling to other services.
- Owns its own data (its own DB or schema) and exposes APIs/events for others.

I avoid splitting based purely on “this part is heavy” and instead focus on stable business boundaries.

```

---

## 3. Scaling, Load Balancing & Availability

### Q8. What does it mean for a system to be “scalable”?

```text
A scalable system can handle increased load (more users, more requests, more data) by adding resources in a predictable way.

In practice:
- If traffic doubles, I can handle it by adding more instances or capacity without rewriting the whole system.
- Performance degrades gracefully rather than collapsing.

```

---

### Q9. What is a load balancer and why do we use it?

```text
A load balancer sits in front of multiple instances of a service and distributes incoming traffic across them.

We use it to:
- Share load between instances (horizontal scaling).
- Improve availability (if one instance fails, traffic goes to others).
- Provide a single stable endpoint (IP/hostname) for clients.

```

---

### Q10. How would you scale a read-heavy API that uses a relational database?

```text
Ideas:
- Add caching in front of the DB (e.g. Redis) for frequently read data.
- Add read replicas for the database and send read traffic to replicas.
- Optimise queries and add proper indexes.
- Paginate responses and avoid returning huge datasets.
- For some cases, precompute or denormalise data to avoid heavy joins.

The goal is to reduce repetitive, expensive reads hitting the primary database.

```

---

### Q11. What’s the difference between **scaling the app layer** and **scaling the database layer**?

```text
Scaling the app layer:
- Running more instances/containers of the backend service.
- Usually done with a load balancer + auto-scaling.
- Often relatively easy because app servers are stateless or lightly stateful.

Scaling the database layer:
- Vertical scaling (bigger DB machine).
- Horizontal approaches (read replicas, sharding/partitioning).
- More complex, because data consistency, transactions, and queries are involved.

```

---

### Q12. How would you increase the availability of a backend service?

```text
Approaches:
- Run multiple instances of the service behind a load balancer.
- Deploy across multiple availability zones or regions if needed.
- Use health checks and automatic restarts for failed instances.
- Remove single points of failure (e.g. have redundant DB and cache setups).
- Implement safe deployment strategies (rolling updates, blue–green) to avoid downtime during releases.

```

---

## 4. Data & Storage Choices (System Design Lens)

### Q13. What factors influence your choice between a single relational database vs multiple databases or services?

```text
Factors:
- Complexity of the domain: is one schema enough or do we have clearly separate domains?
- Team structure: separate teams owning separate services and data.
- Scale: some parts may need different storage technologies (e.g. analytics vs OLTP).
- Isolation and risk: critical domains (payments) might need their own DB for safety/compliance.

I would start with a single relational DB unless there’s a strong reason to split, then separate when scale and domain boundaries justify it.

```

---

### Q14. When might you add a **read replica** or **cache** in front of a database?

```text
I’d add:
- A read replica:
  - When read traffic is high and you want to offload the primary DB.
  - When you need separate workloads (e.g. analytics queries) without impacting writes.

- A cache (e.g. Redis):
  - For very frequent, mostly read-only data.
  - For expensive queries or aggregations where slight staleness is acceptable.

Both are for scaling reads and protecting the primary database.

```

---

### Q15. How do you think about choosing between SQL and NoSQL in a system design?

```text
I tend to default to SQL when:
- Data is structured and relational.
- Transactions and strong consistency matter.
- I need rich querying and joins.

I’d consider NoSQL when:
- The schema is very flexible or evolving rapidly.
- I need very high write throughput or horizontal scale with simpler access patterns.
- The data model (documents, key–value, wide-column) maps naturally to NoSQL.

Often, systems use both where each fits best.

```

---

## 5. Caching in System Design

### Q16. Where in an architecture can caching appear (different layers)?

```text
Caching can appear at several layers:
- Client-side: browser / mobile app caching responses or assets.
- CDN / reverse proxy: caching static assets and sometimes API responses at the edge.
- Application layer: in-memory caches or Redis for computed results or DB query results.
- Database layer: internal DB cache/buffer (handled by the DB engine).

The exact mix depends on the workload and where the bottlenecks are.

```

---

### Q17. When is caching a bad idea or something to be careful with?

```text
Be careful with caching when:
- Data must be strongly consistent (e.g. balances) and stale data would be risky.
- Data changes very frequently, so cache churn might outweigh benefits.
- Data is highly sensitive (you must ensure secure storage and correct scoping).
- Cache invalidation is complex and bugs could lead to user confusion.

Caching is a performance optimisation; correctness must still be handled with the database as the source of truth.

```

---

### Q18. How would you add caching to an endpoint that is slow but read-heavy?

```text
I’d:
1. Identify the slow part (likely a DB query or computation).
2. Introduce a cache (e.g. Redis) with a key that represents the request parameters.
3. On request:
   - Check cache first.
   - If hit → return cached value.
   - If miss → compute/read from DB, store in cache with a TTL, then return.
4. On writes/updates that affect this data, either:
   - invalidate the relevant cache keys, or
   - rely on a short TTL if exact freshness isn’t critical.

That’s essentially the cache-aside pattern.

```

---

## 6. Asynchronous Work, Queues & Events

### Q19. Why might you introduce a message queue (e.g. Kafka, RabbitMQ) into a system?

```text
Reasons:
- Decouple producers and consumers: the caller doesn’t need to wait for work to complete.
- Smooth out traffic spikes by buffering work in a queue.
- Improve reliability: failed jobs can be retried from the queue.
- Enable event-driven features: multiple consumers can react to the same events.

It’s less about being “faster than HTTP” and more about decoupling and resilience.

```

---

### Q20. Give an example of an operation that you would move to an asynchronous background process instead of doing synchronously in a request.

```text
Examples:
- Sending emails or push notifications after a user action.
- Generating PDFs or reports.
- Resizing or transcoding uploaded images/videos.
- Updating analytics counters or logs.

The pattern is: if the user doesn’t need the result immediately to proceed, it’s a good candidate for async processing.

```

---

### Q21. What is the difference between synchronous and asynchronous communication between services?

```text
Synchronous:
- The caller sends a request and waits for a response (e.g. HTTP).
- The caller is blocked until the other service responds or times out.

Asynchronous:
- The caller sends a message/event to a queue or topic and doesn’t wait for immediate processing.
- A consumer processes the message later.
- The caller can continue without being blocked by the downstream service.

Sync is simple and great for request/response APIs; async is useful for decoupling and handling slow or heavy work.

```

---

### Q22. What are some pros and cons of event-driven architectures?

```text
Pros:
- Loose coupling between services.
- Easy to add new consumers that react to events without changing producers.
- Good for workflows where many things react to the same business event.

Cons:
- Harder to reason about overall flow (who is doing what, and when).
- Debugging can be more complex (events flowing through multiple services).
- Data is often eventually consistent, not instantly consistent.
- Requires good observability to understand what’s happening.

```

---

## 7. Reliability, Fault Tolerance & Backpressure

### Q23. What happens if a downstream dependency (e.g. payment service) is slow or failing? How can you protect your system?

```text
If a dependency is slow or failing, requests can pile up and tie up threads, eventually taking down the service.

To protect the system, I can:
- Set timeouts on calls to the dependency.
- Implement retries with backoff (but with limits).
- Use a circuit breaker to stop calling a failing service for a while.
- Return a graceful error or degraded behaviour to the user.
- Offload some work to queues so the main request path stays responsive.
 
```

---

### Q24. What is a circuit breaker pattern, conceptually?

```text
A circuit breaker monitors calls to a downstream service.

- When failures exceed a threshold, it "opens" the circuit:
  - future calls fail fast or use a fallback without even trying the dependency.
- After a delay, it moves to a "half-open" state and lets a few calls through to test if the service has recovered.
- If those succeed, it "closes" again and resumes normal traffic.

It prevents your system from wasting resources on a dependency that is down or very slow.

```

---

### Q25. How would you prevent your system from being overwhelmed by too many incoming requests?

```text
Approaches:
- Rate limiting and quotas at the edge (per user/IP/API key).
- Load shedding: gracefully rejecting some requests when under high load.
- Using queues so that work is buffered and processed at a controlled rate.
- Horizontal scaling of stateless services.
- Backpressure mechanisms in async processing.

The goal is to protect core services and the database so they remain responsive.

```

---

## 8. Security & Multi-Tenancy (High-Level)

### Q26. What are common security considerations you think about when designing an internet-facing API?

```text
Common considerations:
- Authentication and authorization (who are you, what are you allowed to do).
- Input validation and sanitisation (prevent SQL injection, XSS, etc.).
- Use HTTPS everywhere.
- Proper CORS configuration where needed.
- Protect against brute-force and abuse (rate limiting, captcha where appropriate).
- Secure storage of secrets and keys.
- Principle of least privilege in access to DBs and other services.
- Logging and monitoring for suspicious activity.
```

---

### Q27. How would you roughly handle authentication and authorization in a modern backend (high-level only)?

```text
High-level flow:

- Authentication:
  - User logs in with credentials to an auth endpoint.
  - Backend verifies credentials and issues a token (e.g. JWT) or a session.
  - The client sends the token/session with each request.

- Authorization:
  - Each request’s token is validated.
  - The backend checks roles/permissions (RBAC or similar) against what the user is trying to access.
  - Sensitive operations check both identity and ownership (e.g. "is this user allowed to modify this resource?").

```

---

### Q28. In a multi-tenant system (many customers sharing the same app), how might you isolate data between tenants?

```text
Options:

- Logical isolation:
  - Add a tenant_id column to tables.
  - Ensure every query is scoped by tenant_id.
  - Use strict checks in the app layer and possibly row-level security.

- Stronger isolation:
  - Separate schemas per tenant.
  - Or even separate databases per tenant for very high isolation or compliance.

Choice depends on scale, regulatory requirements, and how strongly tenants must be isolated from each other.

```

---

## 9. Observability: Logging, Metrics, Tracing

### Q29. What is “observability” in the context of system design?

```text
Observability is the ability to understand what’s happening inside a system from the outside, using:

- Logs,
- Metrics,
- Traces.

A system is observable if, when something goes wrong or behaves oddly, you can quickly figure out why using the signals it emits.

```

---

### Q30. What would you log in a backend service, and why?

```text
I’d log:

- Request-level information:
  - request ID, endpoint, method, status code, latency.
- Errors and exceptions:
  - stack traces with enough context to debug.
- Important business events:
  - e.g. user signup, payment failure, critical state changes.
- Security-relevant events:
  - login failures, permission denied, suspicious patterns.

Logs should be structured (JSON) and free of sensitive data like passwords or full card numbers.

```

---

### Q31. What kind of metrics would you collect for an API, and what would you do with them?

```text
Metrics:

- Traffic: requests per second, per endpoint.
- Latency: p50/p95/p99 response times.
- Errors: rate of 4xx and 5xx responses.
- Resource usage: CPU, memory, DB connections, queue lengths.

I’d use them to:
- Set up dashboards for visibility.
- Configure alerts when error rate or latency spikes.
- Make scaling decisions and find bottlenecks.

```

---

### Q32. What is distributed tracing and when is it useful?

```text
Distributed tracing tracks a single request as it flows through multiple services.

It attaches a trace ID to the request and records spans (timed segments) in each service.

It’s useful when:
- You have microservices and need to see where time is spent.
- You’re debugging complex flows across several services.
- You want to quickly find which service is causing slow requests.

```

---

## 10. Design Scenarios (Medium-Sized)

### Q33. Design a simple “URL shortener” service (like bit.ly).

At a high level, describe:

* core components,
* data storage,
* how you handle redirects.

```text
Core components:
- HTTP API service:
  - POST /shorten to create short URLs.
  - GET /{short_code} to redirect.

Data storage:
- A simple data store mapping short_code → original_url (could be a relational DB or a key–value store).
- Store creation time, owner, and maybe usage stats.

Flow:
- When a user calls POST /shorten with a URL, the service:
  - generates a unique short_code,
  - stores (short_code, original_url, metadata) in the database,
  - returns the short URL.
- When someone visits /{short_code}, the service:
  - looks up original_url by short_code,
  - optionally tracks a hit,
  - returns an HTTP 301/302 redirect to original_url.

To scale:
- Add caching for short_code lookups (e.g. Redis).
- Put the service behind a load balancer.

```

---

### Q34. Design a basic “news feed” for a social app (just posts from people a user follows).

How would you:

* store the data,
* fetch a user’s feed,
* scale reads?

```text
Storage:
- users table,
- follows table: follower_id, followee_id,
- posts table: id, user_id, content, created_at.

Fetching a feed (simple approach):
- For a user U:
  - find all users U follows,
  - fetch recent posts where user_id is in that list,
  - order by created_at DESC,
  - paginate.

Scaling reads:
- Add proper indexes (on posts.user_id, posts.created_at).
- Add caching: cache the latest feed results per user in Redis.
- For high scale, consider precomputing feeds:
  - fan-out on write: when a user posts, push post IDs into followers’ feed lists.
  - then GET /feed just reads from a per-user sorted list in cache/DB.

Exact strategy depends on write vs read patterns and scale.

```

---

### Q35. Design an endpoint that allows users to upload files (e.g. images or videos).

What components would you use, and how would you scale storage and delivery?

```text
Components:
- Backend API service to handle upload requests and metadata.
- Object storage (e.g. S3, Blob Storage) to store files.
- CDN in front of object storage to serve files efficiently worldwide.

Typical flow:
- Client requests an upload URL.
- Backend generates a pre-signed URL or similar and returns it.
- Client uploads file directly to object storage.
- Backend stores metadata (file URL, owner, size) in a DB.

Scaling:
- Object storage and CDN handle large volume and bandwidth.
- Backend is not a bottleneck because files go directly to storage.
- You can add background processing for thumbnails/transcoding via queues if needed.

```

---

### Q36. Design a simple “notifications” system (e.g. for likes/comments).

How would notifications be generated, stored, and delivered to users?

```text
Generation:
- When an event occurs (like, comment), the service emits an event (e.g. "post_liked", "comment_added").

Storage:
- A notifications service consumes these events.
- It writes notifications to a notifications table:
  - id, user_id (recipient), type, payload (e.g. who liked what), read_flag, created_at.

Delivery:
- For real-time:
  - Use WebSockets or Server-Sent Events to push new notifications to connected clients.
- For offline:
  - Clients can poll an endpoint like GET /notifications.
  - Optionally send email or push notifications for some events.

Scaling:
- Use a queue/event bus between main app and notification service.
- Index notifications.user_id and created_at.
- Optionally cache unread counts.

```

---

### Q37. A client’s API becomes slow when traffic spikes.

What are some architectural changes you would consider to improve performance and resilience?

```text
I’d consider:

- Scaling app servers horizontally behind a load balancer.
- Adding caching for expensive, frequently used endpoints.
- Optimising database access:
  - indexes, query tuning, limiting data returned.
- Introducing queues for heavy or non-critical work so requests don’t block.
- Implementing rate limiting to protect the system from abusive clients.
- Reviewing connection pools and timeouts to avoid resource exhaustion.

```

---

### Q38. A system currently sends emails synchronously when a user performs an action, causing slow responses.

How would you redesign this?

```text
I’d move email sending to an asynchronous flow:

- When the user action occurs, the main service:
  - commits the action to the DB, then
  - enqueues an "send_email" message (e.g. in a message queue).

- A background worker or separate email service:
  - consumes messages from the queue,
  - sends emails using an email provider,
  - handles retries on failure.

The user-facing request returns quickly, and email sending happens in the background.

```

---

## 11. Trade-Off Questions

### Q39. In system design, you often have to choose between **simplicity** and **flexibility/scalability**.

Can you give an example of such a trade-off and how you’d decide?

```text
Example:
- Choosing between a simple monolithic app with one DB vs a microservices setup with multiple services and databases.

Trade-off:
- Monolith:
  - Simpler to develop and operate, but less flexible at very large scale.
- Microservices:
  - More flexible and scalable, but significantly more complex to design and operate.

How I decide:
- Start simple (monolith) when the team is small and requirements are evolving.
- Add complexity (services, queues, multiple DBs) only when there’s a clear scaling or organisational need that justifies it.

```

---

### Q40. When is it worth adding complexity like microservices, queues, and multiple databases, and when is it better to keep things simple?

```text
It’s worth adding complexity when:
- You have concrete scaling or reliability problems that simpler approaches can’t solve.
- Different parts of the system clearly require different scaling or data models.
- The team and organisation can handle the operational overhead.

It’s better to keep things simple when:
- The product is early-stage or small scale.
- The team is small and needs fast iteration.
- The added complexity doesn’t provide clear, measurable benefits yet.

In general, I prefer to start simple, monitor, and then evolve the architecture as real needs appear.

```

---

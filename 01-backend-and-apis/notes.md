# 01 – Backend & APIs

This module is about **understanding and explaining** backend & API fundamentals, with small code snippets as support (not full apps).

---

## 🎯 What I should be able to explain (high-level checklist)

By the time I’m done with this module, I should be able to:

### HTTP & REST
- [ ] Explain the main HTTP methods: GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD  
- [ ] Explain **safe** vs **idempotent** methods (GET/HEAD/OPTIONS are safe; PUT/DELETE are idempotent, POST is not)  
- [ ] Know common status codes and when to use them:
  - 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500  
- [ ] Explain what a **resource** and **URI** are in REST terms  
- [ ] Design a simple REST API for a basic domain (e.g. tasks, posts, users)

### API Design
- [ ] Decide when to use **path params** vs **query params** vs body  
- [ ] Explain pagination (limit/offset or cursor-based)  
- [ ] Explain filtering & sorting  
- [ ] Show how I would structure request/response schemas  
- [ ] Explain how I’d do **validation** & **error handling**

### Auth & Security
- [ ] Explain **authentication** vs **authorization**  
- [ ] Explain difference between **sessions** and **JWT-based auth**  
- [ ] Explain why we hash passwords, and name at least one algorithm (bcrypt/argon2)  
- [ ] Explain how I would protect an API endpoint in theory  
- [ ] Name a few common web security concerns: SQL injection, XSS, CSRF, insecure storage, misconfigured CORS

### WebSockets & Realtime
- [ ] Explain how WebSockets differ from plain HTTP  
- [ ] Name use cases: chat, notifications, live dashboards, collaborative editing  
- [ ] Outline when I’d pick WebSockets vs polling vs Server-Sent Events (SSE)

---


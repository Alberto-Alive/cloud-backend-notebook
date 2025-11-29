# 01 – Backend & APIs

This module is about **understanding and explaining** backend & API fundamentals, with small code snippets as support (not full apps).

---

## 🎯 What I should be able to explain (high-level checklist)

By the time I’m done with this module, I should be able to:

### HTTP & REST
- [x] Explain the main HTTP methods: GET, POST, PUT, PATCH, DELETE, OPTIONS, HEAD  
- [x] Explain **safe** vs **idempotent** methods (GET/HEAD/OPTIONS are safe; PUT/DELETE are idempotent, POST is not)  
- [x] Know common status codes and when to use them:
  - 200, 201, 204, 400, 401, 403, 404, 409, 422, 429, 500  
- [x] Explain what a **resource** and **URI** are in REST terms  
- [x] Design a simple REST API for a basic domain (e.g. tasks, posts, users)

### API Design
- [x] Decide when to use **path params** vs **query params** vs body  
- [x] Explain pagination (limit/offset or cursor-based)  
- [x] Explain filtering & sorting  
- [x] Show how I would structure request/response schemas  
- [x] Explain how I’d do **validation** & **error handling**

### Auth & Security
- [x] Explain **authentication** vs **authorization**  
- [x] Explain difference between **sessions** and **JWT-based auth**  
- [x] Explain why we hash passwords, and name at least one algorithm (bcrypt/argon2)  
- [x] Explain how I would protect an API endpoint in theory  
- [x] Name a few common web security concerns: SQL injection, XSS, CSRF, insecure storage, misconfigured CORS

### WebSockets & Realtime
- [x] Explain how WebSockets differ from plain HTTP  
- [x] Name use cases: chat, notifications, live dashboards, collaborative editing  
- [x] Outline when I’d pick WebSockets vs polling vs Server-Sent Events (SSE)

---


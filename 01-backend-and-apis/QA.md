
---

## 1️⃣ HTTP & REST

### Concepts & Definitions

1. What is HTTP, and how does it relate to REST APIs?
HTTP is an application-layer protocol for client-server communication over the web.
REST is an architectural style for designing APIs, and REST APIs typically use HTTP as the trasnport, using HTTP methods (GET, POST, etc) to operate on resources.

2. Explain the difference between the following HTTP methods:

   * GET : retrieve a resource (safe, idempotent). 
   * POST : create or trigger an action (unsafe, not idempotent).
   * PUT : fully replace a resource (unsafe, idempotent).
   * PATCH : partially update a resource (unsafe, not quaranteed idempotent ~ ex counter increment)
   * DELETE : delete a resource (unsafe, idempotent in theory ~ ex delete child row)
   * OPTIONS : describe communication options (safe, often used for CROS)
   * HEAD : like GET but only headers, no body (safe, idempotent).

3. What does it mean for an HTTP method to be **safe**? Which methods are considered safe? safe it means it does not change the resource on the server: GET, OPTIONS, HEAD
4. What does it mean for an HTTP method to be **idempotent**? Which methods are idempotent? idempotent means no matter how many times you call that HTTP method the resource is not changed beyong the first time the method was called; idempotent(get, head, options, delete, put) not-idempotent(post, patchc aka popa)
5. Why is **POST** not considered idempotent? because every time is called a new resource is created
6. In REST, what is a **resource**? Give two concrete examples. in rest a resource is a conceptual thing you can access via a URI: e.g. a user /users/123, a blog post /posts/43, a collection of tasks /tasks
7. What is a **URI** and how is it used to represent resources in REST? uri stands for uniform resource identifier and it is used to uniquely target rest api resources; it includes path and optional query string: /users/123?include=posts
8. What is the difference between a **URI**, **URL**, and **URN** (at a high-level)? 
  uri
url urn
terms:
uri : id
url: location
urn: name
uri, url, urn: uniform resource
: uri contains both url and urn
9. What is the typical structure of an HTTP request (main parts)?
- 1. request line: method + path + HTTP version (GET /api/users/1 HTTP/1.1)
- 2. headers: metadata (Host, Content-Type, Authorization)
- 3. Optional body: e.g. JSON payload for POST/PUT/PATCH
10. What is the typical structure of an HTTP response (main parts)?
- 1. status line - HTTP version + status code + reason phrase e.g. HTTP/1.1 200 OK
- 2. Headers - Content-type, content-length
- 3. optional body - the response payload (JSON/HTML/etc.)

### Status Codes

11. When would you return a **200 OK** response? Give two examples. when a resource was retrieved successfully (retrieved user details GET /user), when a resource was updated successfully (PATCH /users/{user_id}).
12. When would you use **201 Created**? What extra header is often included with it? i'd use 201 when creating a new user, location could be an extra header
13. When would you use **204 No Content** instead of **200 OK**? The core idea of 204: “Request succeeded, but no response body”. Successful DELETE where you don’t return a body.
Successful PUT/PATCH where client doesn’t need a representation.
14. What is the difference between **400 Bad Request** and **422 Unprocessable Entity**?400 Bad Request – the request is malformed or invalid (bad JSON, missing required fields, invalid query string).

422 Unprocessable Entity – the request is syntactically correct, but semantically invalid (e.g. JSON is valid but violates business rules/validation).
15. When would an API return **401 Unauthorized** vs **403 Forbidden**?401 Unauthorized = “You are not authenticated (or your token is invalid).”
→ Usually: send WWW-Authenticate header, ask to log in.

403 Forbidden = “I know who you are, but you’re not allowed to do this (no permission).”
16. What are typical situations where you’d return **404 Not Found**? client is trying to access non-existent resource
17. When is **409 Conflict** appropriate? Give an example scenario. Trying to create a username that already exists.

Updating a resource with an outdated version number (optimistic locking).

Better: 409 is used when the request conflicts with the current state of the resource.
18. What are common cases where you’d return **429 Too Many Requests**? when the clients exceeds the request limit set by the dev
19. When should you return **500 Internal Server Error**? 500 means: the server is up, but it hit an unexpected error while processing this request (bug, unhandled exception, DB crash mid-request, etc.).
20. Given a scenario, choose the best status code:

    * A user tries to delete a resource that does not exist.  404
    * A user sends malformed JSON in the request body. 400
    * A user tries to create a duplicate username. 409

### Design a Simple REST API

21. Design a basic REST API for a **Task** resource. What URIs and methods would you define for:

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
from typing import List

app = FastAPI()

class TaskCreate(BaseModel):
  title: str
  description: str

class Task(TaskCreate):
  id: int

tasks: List(Task) = []
next_id = 1


    * Create task
@app.post("/tasks", response_model = Task, status_code=201)
def create_task(task: str):
  return task
    * List tasks
@app.get("/tasks")
def list_tasks():
  return tasks
    * Get a single task
@app.get("/tasks")
def get_one_task(task_id: int):
  for task in tasks:
    if task.id == task_id:
      return task
  raise HTTPException(status_code=404, detail="Task not found")
    * Update a task
@app.put("tasks/{task_id}", response_model=Task)
def update_task(task_id: int, payload: TaskCreate):
  for i, task in enumerate(tasks):
    if task.id == task_id:
      update = Task(id=task_id, **payload.dict())
      tasks[i] = updated
      return updated
  raise HTTPException(status_code= 404, detail="Item not found")
    * Delete a task
@app.delete("tasks/{task_id}", status_code=204)
def delete_task(task_id: int):
  for i, task in enumerate(tasks):
    if task.id == task_id:
      tasks.pop(i)
      return
  raise HTTPException(status_code=404, detail="Task not found")
```
22. For a **User** resource, would you expose `/users/login` as a POST endpoint? Why or why not? Any alternatives?
Well, probably not, I'd go with POST /session
23. For a **Post** and its **Comments**, how would you design the URIs (e.g. nested routes)?
Starting with posts:
list post: get /posts
create posts: post /posts
get one post: get /posts/{post_id}
update post: put or patch /posts/{post_id}
delete post: delete /posts/{post_id}

Let's do comments by a specific post:
list comments for a sepcific psot: get /posts/{post_id}/comments
create a new comment for that post: post /posts/{post_id}/comments
get a single comment under that post: get /posts/{post_id}/comments/{comment_id}
update a comment under a post: PUT or PATCH /posts/{post_id}/comments/{comment_id}
delete a comment: delete /posts/{post_id}/comments/{comment_id}

### Small Snippet Questions

24. Given this HTTP request line:

```http
GET /api/tasks/123 HTTP/1.1
```

* What is the HTTP method? get
* What is the resource? task 123
* What would a successful response status code likely be? 200

25. You receive:

```http
HTTP/1.1 201 Created
Location: /api/users/42
```

* What just happened on the server? a new record created successfully
* What does the `Location` header tell the client? returns the location where the user lives

---

## 2️⃣ API Design

### Params & Body

26. When should you use **path params** vs **query params** vs **request body**? Give examples for each.
path params: specifies where the resource lives: /users /comments{comment_id} /posts/76
query params: filters: /users?limit=20&offset=40
request body: sending or updating resources POST /users {"email": "github@email.com", "pass": "123"}

Path Params = identity
Query Params = modifiers
Body = data

27. For a `GET /products` endpoint:

    * Which parts would you put in path, which in query?

      * Category - query
      * Page number - query
      * Product ID - path
      * Search term - query

28. Is it appropriate to send a complex JSON object in the query string? Why or why not?
no, is not maintainable and URLs have length limits
### Pagination

29. Explain **limit/offset pagination**. How would a client specify page size and page number? limit=20&offset=40 limits how many records to return and offsets how many records to skip before returning those records
30. Explain **cursor-based pagination**. When is it preferable over limit/offset? because cursor-based pagination uses an id to the next set of records works better for apps when say new records are introduced
31. Given an endpoint `GET /api/posts`, design a pagination scheme using limit/offset (query parameters).
GET /api/posts?limit=20&offset40
32. What are pros and cons of limit/offset pagination?
easy to implement and understand but slow for large offsets and not stable when data changes as it can cause duplicates or missing items

### Filtering & Sorting

33. How would you add **filtering** to `GET /api/users`? Give example query params.
get /api/users?age_lt22
34. How would you add **sorting** to `GET /api/posts`? Show an example query string that sorts by `createdAt` descending.
get /api/posts?sort=-createdAt
35. How would you combine pagination, filtering, and sorting in a single request?
GET /api/posts?limit=20&offset=40&author=john&minLikes=100&sort=-createdAt


### Schemas, Validation & Errors

36. What is a **request schema**? What is a **response schema**?
request schema is the expected format for a request likeswise the response is for a response
37. Why is it beneficial to explicitly define schemas (e.g. with JSON Schema, Zod, Joi, etc.)?
to ensure no extra fields are inserted to affect the security, all the fields are present with the right data type
38. Given this JSON body for `POST /api/tasks`:

```json
{
  "title": "Buy milk",
  "description": "2L of whole milk",
  "dueDate": "2025-12-01T10:00:00Z"
}
```

* Which fields are required? title
* What types would you give each field? str str datetime

39. What kind of validation would you perform on a user registration payload:

```json
{
  "email": "user@example.com",
  "password": "123456",
  "age": 12
}
```

What status code(s) might you return for invalid data, and what would the error response look like?

40. How would you structure an error response so that the frontend can:

    * Show a generic message
    * Show field-specific validation errors

    (Describe the JSON structure.)

    {
      "error":{
        "message": "The server returned an error",
        "code": "VALIDATION_ERR",
        "details" :{
          "email":["Email is invalid"],
          "password": ["Password must be at least 8 characters"],
          "age" : ["Must be 18 yo or older"]
        }
      }
    }

41. What is the difference between **client-side validation** and **server-side validation**? Why do you still need server-side validation even if client-side is perfect?

client-side validation helps user introduce data correctly whereas server-side validation ensures if m,alicious user introuidced bad data it doesn't pass

---

## 3️⃣ Auth & Security

### Auth vs Authz

42. Explain the difference between **authentication** and **authorization** with a simple example.
43. In an API, which check should typically happen first: authentication or authorization? Why?

### Sessions vs JWT

44. How does **session-based authentication** work in a typical web app?
45. Where is the session ID typically stored on the client side?
46. What is a **JWT (JSON Web Token)** and what is it used for?
47. What are the main differences between **session-based auth** and **JWT-based auth**?
48. What are typical pros/cons of JWT vs sessions (e.g., scaling, revocation, storage)?
49. How would you protect a route like `GET /api/me` using JWTs? Describe the steps the server takes on each request.

### Passwords & Hashing

50. Why should passwords never be stored in plain text? they can be sniffed.. if db breached pass is exposed
51. What does it mean to **hash** a password? Encryption = reversible

Hashing = one-way, irreversible
52. What is a **salt** and why is it important?
A salt is a random value added to the password before hashing, and stored alongside the hash.

Why it's important:

Prevents rainbow table attacks

Ensures identical passwords produce different hashes

Prevents attackers from spotting users with the same password

Slows down brute-force attacks
53. Name at least one password hashing algorithm you would use in production (e.g. bcrypt, argon2). Why is it preferred over simple hashing like SHA-256? bcrypt is preferred because is more secure
These algorithms are better than SHA-256 because they are:

Slow on purpose → makes brute-force attacks expensive

Memory-hard (argon2) → prevents GPU attacks

Include built-in salting

Designed specifically for passwords
54. Describe the steps of signing up a user and logging in a user securely (high level).
Signup (POST /users or POST /signup)

Client sends { email, password }

Server:

Validates input

Generates a salt

Hashes the password using bcrypt/argon2

Stores email + hashedPassword + salt in DB

Server returns 201 Created


Login (POST /sessions or POST /auth/login)

Client sends { email, password }

Server:

Finds user by email

Hashes given password using stored salt

Compares hash with stored hash

If valid:

Creates a session or JWT

Returns token (e.g., "accessToken": "...")

If invalid:

Return 401 Unauthorized

### Protecting Endpoints

55. How would you protect a `GET /api/admin/users` endpoint so only admins can access it? Describe the checks involved.
check user jwt or session if allows for visiting that path, protect that path in main.py
56. How can you ensure that an API endpoint that modifies user data can only be used by the owner of that data? always verify if the client is the owner of the data
57. Where would you typically perform auth checks in a backend framework (e.g., middleware vs inside the handler)? authentication is typically done in middleware so every request can be checked. Authorization (role/permission/ownership checks) can also be done in middleware or in route-level decorators, depending on how fine-grained the rules are.

### Common Web Security Risks

58. What is **SQL injection**? Give a basic example and explain how to prevent it.
when an attacker injects malicious SQL into a query because user input is not properly sanitized or parameterized.
59. What is **XSS (Cross-Site Scripting)**? How can it affect users of your app?
XSS is about injecting JavaScript into your app to run in the victim’s browser.60. What is **CSRF (Cross-Site Request Forgery)**? How could it be used against an authenticated user?
tricking an authenticated user’s browser into sending unintended requests to your server.
61. What is **CORS** and what problem does it solve?
CORS = Cross-Origin Resource Sharing

It defines which websites are allowed to make browser requests to your API.

Without CORS:

Browsers block JS requests to different origins

CORS allows controlled cross-origin communication
62. What are some examples of **insecure storage** of sensitive data?
Examples:

Storing passwords in plain text

Storing passwords with fast hashing (MD5, SHA-1, SHA-256)

Storing tokens or credentials unencrypted on disk

Logging sensitive data (passwords, tokens)

Storing secrets in GitHub repos

Storing API keys in frontend code

Storing PII without encryption at rest
63. What is a **misconfigured CORS** policy, and how can it become a security risk?
usually what you do in templating:)) 
Access-Control-Allow-Origin: *
Access-Control-Allow-Credentials: true

64. Name at least three practices to improve the security of a REST API.
Validate all inputs (schemas)

Use HTTPS everywhere

Hash and salt passwords (bcrypt/argon2)
---

## 4️⃣ WebSockets & Realtime

### WebSockets Basics

65. How do WebSockets differ from plain HTTP?
Create a persistent, bidirectional connection
66. Describe the lifecycle of a WebSocket connection (open, message, close).
  1. Handshake

  Client sends a special HTTP Upgrade request

  Server agrees to upgrade to WebSocket protocol (101 Switching Protocols)

  2. Open

  Connection established

  Both sides can start sending messages

  3. Message exchange

  Client and server send messages asynchronously

  No request/response — pure event-based communication

  4. Close

  Either side can initiate a close frame

  Close handshake happens

  Connection is terminated cleanly
67. What are some examples of data that are good candidates for realtime updates?
notifications, messages, comments, likes, 

### Use Cases

68. Why are WebSockets a good fit for:

    * Chat applications: WebSockets provide a persistent, bidirectional connection, so both clients and server can send messages instantly without polling.
    * Live notifications: WebSockets allow the server to push new events to the client the moment they occur, without the client constantly asking.
    * Collaborative editors: WebSockets let every change sync instantly to all connected clients.
    * Live dashboards / trading tickers: Dashboards and stock tickers require rapid, continuous, real-time data updates.

69. For each of the above, what would be the downside of using only plain HTTP requests?
way to many call for each action: Using only plain HTTP requests (especially polling) for real-time features has major downsides:


### WebSockets vs Polling vs SSE

70. What is **polling**? How does it work with HTTP? server responds with current non-updated data for a number of client requests
71. What is **long polling** and how does it differ from basic polling?
Long polling = the client sends a request, and the server keeps the request open until new data is available.
Difference from basic polling:

Basic polling → request every X seconds

Long polling → one request stays open until an event occurs
72. What are **Server-Sent Events (SSE)**? How are they different from WebSockets?
SSE (Server-Sent Events) = a one-way realtime connection from server → client over HTTP.

Key points:

Client makes a single request: GET /events

Server keeps connection open

Server can continuously push events

Uses HTTP (not a separate protocol)

Automatic reconnection built in
73. When would you choose **WebSockets** over polling?
Polling cannot match the performance or responsiveness of WebSockets.
74. When would **SSE** be a better choice than WebSockets?
You only need server → client updates (one-way)
75. For a simple “notification bell” in a web app that shows realtime notifications, what strategy would you pick (polling, SSE, or WebSockets) and why?
websockets because users would feel more connected to the app


### Small Scenario Questions

76. You’re building:

    * A live chat between users: websockets
    * A dashboard that updates metrics every 10 seconds: sse
    * An email sending status page (status changes a few times, then done): sse

    For each case, choose between: basic polling, long polling, SSE, or WebSockets and explain your choice.

---

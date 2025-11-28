## 2. Designing REST APIs

### 2.1 Resources & naming

Principles: use nouns for resources:  /users, /videos, /orders
            use plural for collections: /users/{id}
            avoid verbs in URLs
Examples:

GET /posts - list posts
POST /posts - create posts
GET /posts/{post_id} - get one post
PATCH /posts/{post_id} - partial update
DELETE /posts/{post_id} - delete post
GET /posts/{post_id}/comments - list comments for a post
POST /posts/{post_id}/comments - add comment to a post


### 2.2 Path, query, and body

PATH params: identity of a resource (/users/{user_id})
Query params: filtering, sorting, pagination (/users?limit=20&offset=40)
Body: complex input, usually JSON

```python

from typing import Optional, List
from fastapi import Query
from pydantic import BaseModel

class Post(BaseModel):
    id: int
    title: str
    body: str

POSTS = [
    Post(id=1, title="First", body="..."),
    Post(id=2, title="Second", body="..."),
]

@app.get("/posts", response_model=List[Post])
def list_posts(
    limit: int = Query(10, ge=1, le=100),
    offset: int = Query(0, ge=0),
    search; Optional[str] = None,
)

results = POSTS
if search:
    results = [p for p in POSTS if search.lower() in p.title.low()]
return results[offset : offset + limit]
```
### 2.3 Versioning

URI Versioning: /v1/users, /v2/users
API versioning is parallel-compatible as in you can run old versions (v1) and new versions (v2) at the same time
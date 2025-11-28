## 1. HTTP & REST Fundamentals

### 1.1 HTTP methods

**Key ideas:**

- **GET** – read data, no side effects (safe, idempotent)
- **POST** – create or trigger actions, not idempotent by default
- **PUT** – replace an entire resource, idempotent
- **PATCH** – partial update, not guranteed idempotent
- **DELETE** – remove a resource, idempotent  
- **HEAD** –  like GET but headers only
- **OPTIONS** – what methods are supported



```python

from fastapi import FastAPI

app = FastAPI()


@app.get("/items")
def list_items():
    return[{"id": 1, "name": "Item 1"}]


@app.post("/items", status_code=201)
def create_item(item: dict):
    return {"id": 2, **item}


@app.put("/items/{item_id}")
def replace_item(item_id: int, item: dict):
    return {"id": item_id, **item}

@app.delete("/items/{item_id}", status_code=204)
def delete_item(item_id: int):
    return None
```

### 1.2 Status codes

- 200 OK - successful read/operation
- 201 Created - resource created (often with Location header)
- 204 No Content - success but no response body (DELETE, idempotent operations)
- 400 Bad Requests - client error, invalid input
- 401 Unauthorized - auth required / invalid credentials
- 403 Forbidden - authenticated but not allowed
- 404 Not Found - resource doesn't exist
- 409 Conflict - conflicting state (e.g. duplicate unique field)
- 422 unprocessable Entity - validation failed
- 429 Too Many Requests - rate limiting
- 500 Internal Server Error - unexpected server error


```python
from fastapi import HTTPException, status

def get_item_or_404(items: dict, item_id:int):
    try:
        return items[item_id]
    except KeyError:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="Item not found",
        )
```

### 1.3 Safe & idempotent

Safe methods (read-only): useful because clients, proxies, CDNs, browsers, and load balancers all assume that safe methods can be repeated, cached retried, and pre-fetched (say on mouse hover pre-fetch the link's resource) without causing damage.
- GET
- HEAD
- OPTIONS

Unsafe methods:
- POST
- PUT
- PATCH
- DELETE


! idempotent: calling the endpoint multiple times does not change the resource beyond the first time.

! un/safe: changes or doesn't change the resource on the server

Example explanation: DELETE /items/1 is idempotent: first call deletes it; further calls do nothing, but the end state is still "item 1 does not exist"
## 3. Data Modeling & Validation

### 3.1 Request vs response models

What we receive from the client has a different model to what we return back to the client.

```python

from datetime import datetime
from typing import Optional
from pydantic import BaseModel, Field

# what we receive from the client
class PostCreate(BaseModel):
    title: str = Field(..., min_length=1, max_length=200) 
    body: str = Field(..., min_length=1)
    published: bool = False

# what we return to the client

class PostOut(BaseModel):
    id: int
    title: str
    body: str
    published: bool
    created_at: datetime

    class Config:
        orm_mode = True

```

SQLAlchemy Core: Turn Python into SQL
SQLAlchemy ORM: Turn database rows into Python objects


```
Request JSON → Pydantic → Python object → SQLAlchemy ORM → Database
Database row → ORM object → Pydantic model → JSON Response
```

Data types: 

| Type   | Display |
| ------ | ------- |
| dict   | `{...}` |
| list   | `[...]` |
| tuple  | `(...)` |
| set    | `{...}` |
| object | `<...>` |


### 3.2 Validation

- validate tpes (strings, ints, dates,..)
- validate ranges and formats (min/max, regex)
- provide clear error messages for clients

FastAPI automatically returns 422 with a structured error body when validation fails (unprocessable Entity - validation failed)
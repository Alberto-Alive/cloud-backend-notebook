## 4. Error Handling & API Shape

Good APIs have predictable error shapes. It's also called error envelope or error schema.

```json
{
  "error": {
    "type": "validation_error",
    "message": "Invalid payload",
    "details": [
      {
        "field": "title",
        "issue": "must not be empty"
      }
    ]
  }
}
```


Real-world API error designing: 

File: app/schemas/errors.py

```python

from typing import Optional, List
from pydantic import BaseModel


class ErrorDetail(BaseModel):
    field: Optional[str]
    issue: str

class ErrorResponse(BaseModel):
    type: str
    message: str
    details: Optional [List[ErrorDetail]] = None

```
ErrorDetail = one line in a list of problems

ErrorResponse = the whole report


Example of custom exception handler:


```python
from fastapi import Request
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
async def validation_exception_handler(request: request, exc: RequestValidationError):
    return JSONResponse(
        status_code=422,
        content={
            "error":{
                "type": "validation_error",
                "message": "Invalida request",
                "details" : exc.errors(),
            }
        }
    )


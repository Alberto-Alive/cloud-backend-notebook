## 5. Authentication & Security


### 5.1 Concepts

Authentication (authN): "Who are you?" - Authentication is the process of verifying a user's identity.

Authorization (authZ): "What are you allowed to do?" - Authorization checks permissions after authentication.

Session vs Tokens (JWT)

They both solve the same problem i.e. keep user logged in but jwt would be better for serverless like unyleague.com and session for the actual app... 



### 5.2 Password hashing & JWT


```python

from datetime import datetime, timedelta
from typing import Optional

from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from jose import jwt, JWTError
from passlib.context import CryptContext

SECRET_KEY = "change-me"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="auth/login")

def hash_password(password: str) -> str:
    return pwd_context.hash(password)


def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)


def create_access_token(data: dict, expires_delta: Optional[timedelta] = None) -> str:
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)


def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        # Here, normally fetch user from DB
        return {"username": username}
    except JWTError:
        raise credentials_exception
```

..then protect an endpoint

```python
@app.get("/me")
def read_me(current_user: dict = Depends(get_current_user)):
    return current_user
```
Flow:
    1. User posts credentials to /auth/login

    2. Server verifies password, creates JWT with sub=<user id>

    3. Client stores token (header or cookie)

    4. Subsequent requests include token, server verifies and loads user
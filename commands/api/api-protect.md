---
description: Add authentication, authorization, and security to API endpoints
model: claude-sonnet-4-5
polyglot: true
---

Add comprehensive security, authentication, and authorization to the specified API route.

## Target API Route

$ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Security Layers to Implement

### 1. **Authentication** (Who are you?)
- Verify user identity
- Token validation (JWT, session, API keys)
- Handle expired/invalid tokens

### 2. **Authorization** (What can you do?)
- Role-based access control (RBAC)
- Resource-level permissions
- Check user ownership

### 3. **Input Validation**
- Sanitize all inputs
- SQL/NoSQL injection prevention
- XSS prevention

### 4. **Rate Limiting**
- Prevent abuse
- Per-user/IP limits
- Sliding window algorithm

---

## Python Stack (py) - FastAPI

### JWT Authentication

```python
# src/core/security.py
from datetime import datetime, timedelta
from typing import Annotated

from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from jose import JWTError, jwt
from pydantic import BaseModel

from src.config import settings

security = HTTPBearer()


class TokenData(BaseModel):
    """JWT token payload."""

    user_id: str
    email: str
    role: str
    exp: datetime


def create_access_token(data: dict, expires_delta: timedelta | None = None) -> str:
    """Create JWT access token."""
    to_encode = data.copy()
    expire = datetime.utcnow() + (expires_delta or timedelta(hours=24))
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, settings.SECRET_KEY, algorithm="HS256")


def verify_token(token: str) -> TokenData:
    """Verify and decode JWT token."""
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=["HS256"])
        return TokenData(**payload)
    except JWTError as e:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
            headers={"WWW-Authenticate": "Bearer"},
        ) from e


async def get_current_user(
    credentials: Annotated[HTTPAuthorizationCredentials, Depends(security)],
) -> TokenData:
    """Get current user from JWT token."""
    return verify_token(credentials.credentials)
```

### Role-Based Authorization

```python
# src/core/permissions.py
from enum import Enum
from functools import wraps
from typing import Callable

from fastapi import HTTPException, status


class Role(str, Enum):
    """User roles."""

    ADMIN = "admin"
    USER = "user"
    GUEST = "guest"


def require_role(*allowed_roles: Role) -> Callable:
    """Decorator to require specific roles."""

    def decorator(func: Callable) -> Callable:
        @wraps(func)
        async def wrapper(*args, **kwargs):
            # Get current_user from kwargs (injected by Depends)
            current_user = kwargs.get("current_user")
            if not current_user:
                raise HTTPException(
                    status_code=status.HTTP_401_UNAUTHORIZED,
                    detail="Authentication required",
                )

            if current_user.role not in [role.value for role in allowed_roles]:
                raise HTTPException(
                    status_code=status.HTTP_403_FORBIDDEN,
                    detail="Insufficient permissions",
                )

            return await func(*args, **kwargs)

        return wrapper

    return decorator
```

### Protected Route Example

```python
# src/api/routes/protected.py
from typing import Annotated

from fastapi import APIRouter, Depends, HTTPException, status

from src.core.permissions import Role, require_role
from src.core.security import TokenData, get_current_user

router = APIRouter(prefix="/protected", tags=["protected"])


@router.get("/profile")
async def get_profile(
    current_user: Annotated[TokenData, Depends(get_current_user)],
):
    """Get current user profile (any authenticated user)."""
    return {"user_id": current_user.user_id, "email": current_user.email}


@router.get("/admin")
@require_role(Role.ADMIN)
async def admin_only(
    current_user: Annotated[TokenData, Depends(get_current_user)],
):
    """Admin-only endpoint."""
    return {"message": "Welcome, admin!", "user": current_user.email}


@router.delete("/users/{user_id}")
async def delete_user(
    user_id: str,
    current_user: Annotated[TokenData, Depends(get_current_user)],
):
    """Delete user (owner or admin only)."""
    is_owner = current_user.user_id == user_id
    is_admin = current_user.role == Role.ADMIN.value

    if not (is_owner or is_admin):
        raise HTTPException(
            status_code=status.HTTP_403_FORBIDDEN,
            detail="Cannot delete other users",
        )

    # Delete logic here
    return {"deleted": user_id}
```

### Rate Limiting with Redis

```python
# src/core/rate_limit.py
import time
from typing import Annotated

from fastapi import Depends, HTTPException, Request, status
from redis.asyncio import Redis

from src.clients.redis_client import get_redis


class RateLimiter:
    """Sliding window rate limiter."""

    def __init__(self, requests: int, window: int):
        self.requests = requests
        self.window = window  # seconds

    async def __call__(
        self,
        request: Request,
        redis: Annotated[Redis, Depends(get_redis)],
    ) -> None:
        """Check rate limit."""
        # Use IP or user ID as key
        client_ip = request.client.host
        key = f"rate_limit:{client_ip}"

        current_time = int(time.time())
        window_start = current_time - self.window

        # Remove old entries
        await redis.zremrangebyscore(key, 0, window_start)

        # Count requests in window
        request_count = await redis.zcard(key)

        if request_count >= self.requests:
            raise HTTPException(
                status_code=status.HTTP_429_TOO_MANY_REQUESTS,
                detail="Rate limit exceeded",
                headers={"Retry-After": str(self.window)},
            )

        # Add current request
        await redis.zadd(key, {str(current_time): current_time})
        await redis.expire(key, self.window)


# Usage: 100 requests per minute
rate_limit = RateLimiter(requests=100, window=60)
```

---

## TypeScript Stack (ts) - Next.js

### JWT Authentication

```typescript
// lib/auth.ts
import { jwtVerify, SignJWT } from 'jose'
import { cookies } from 'next/headers'
import { NextRequest, NextResponse } from 'next/server'

const SECRET = new TextEncoder().encode(process.env.JWT_SECRET!)

interface TokenPayload {
  userId: string
  email: string
  role: string
}

export async function createToken(payload: TokenPayload): Promise<string> {
  return new SignJWT(payload)
    .setProtectedHeader({ alg: 'HS256' })
    .setExpirationTime('24h')
    .sign(SECRET)
}

export async function verifyToken(token: string): Promise<TokenPayload | null> {
  try {
    const { payload } = await jwtVerify(token, SECRET)
    return payload as TokenPayload
  } catch {
    return null
  }
}

export async function getCurrentUser(
  request: NextRequest
): Promise<TokenPayload | null> {
  const token = request.headers.get('authorization')?.replace('Bearer ', '')

  if (!token) {
    return null
  }

  return verifyToken(token)
}
```

### Role-Based Authorization

```typescript
// lib/permissions.ts
import { NextRequest, NextResponse } from 'next/server'
import { getCurrentUser } from './auth'

export type Role = 'admin' | 'user' | 'guest'

interface AuthOptions {
  roles?: Role[]
  requireAuth?: boolean
}

export function withAuth(
  handler: (request: NextRequest, user: any) => Promise<NextResponse>,
  options: AuthOptions = { requireAuth: true }
) {
  return async (request: NextRequest): Promise<NextResponse> => {
    const user = await getCurrentUser(request)

    // Check authentication
    if (options.requireAuth && !user) {
      return NextResponse.json(
        { error: 'Authentication required', success: false },
        { status: 401 }
      )
    }

    // Check roles
    if (options.roles && user && !options.roles.includes(user.role as Role)) {
      return NextResponse.json(
        { error: 'Insufficient permissions', success: false },
        { status: 403 }
      )
    }

    return handler(request, user)
  }
}
```

### Protected Route Example

```typescript
// app/api/protected/profile/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { withAuth } from '@/lib/permissions'

export const GET = withAuth(async (request, user) => {
  return NextResponse.json({
    data: {
      userId: user.userId,
      email: user.email,
      role: user.role,
    },
    success: true,
  })
})
```

```typescript
// app/api/admin/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { withAuth } from '@/lib/permissions'

export const GET = withAuth(
  async (request, user) => {
    return NextResponse.json({
      message: 'Welcome, admin!',
      user: user.email,
      success: true,
    })
  },
  { roles: ['admin'] }
)
```

### Rate Limiting with Upstash Redis

```typescript
// lib/rate-limit.ts
import { Ratelimit } from '@upstash/ratelimit'
import { Redis } from '@upstash/redis'
import { NextRequest, NextResponse } from 'next/server'

const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
})

const ratelimit = new Ratelimit({
  redis,
  limiter: Ratelimit.slidingWindow(100, '1 m'), // 100 requests per minute
  analytics: true,
})

export async function checkRateLimit(
  request: NextRequest
): Promise<{ success: boolean; limit: number; remaining: number }> {
  const ip = request.headers.get('x-forwarded-for') ?? 'anonymous'
  const { success, limit, remaining } = await ratelimit.limit(ip)

  return { success, limit, remaining }
}

export function withRateLimit(
  handler: (request: NextRequest) => Promise<NextResponse>
) {
  return async (request: NextRequest): Promise<NextResponse> => {
    const { success, remaining } = await checkRateLimit(request)

    if (!success) {
      return NextResponse.json(
        { error: 'Rate limit exceeded', success: false },
        {
          status: 429,
          headers: {
            'X-RateLimit-Remaining': remaining.toString(),
            'Retry-After': '60',
          },
        }
      )
    }

    const response = await handler(request)
    response.headers.set('X-RateLimit-Remaining', remaining.toString())
    return response
  }
}
```

### Input Validation with Zod

```typescript
// lib/validation.ts
import { z } from 'zod'
import { NextRequest, NextResponse } from 'next/server'

export function withValidation<T extends z.ZodSchema>(
  schema: T,
  handler: (request: NextRequest, data: z.infer<T>) => Promise<NextResponse>
) {
  return async (request: NextRequest): Promise<NextResponse> => {
    try {
      const body = await request.json()
      const data = schema.parse(body)
      return handler(request, data)
    } catch (error) {
      if (error instanceof z.ZodError) {
        return NextResponse.json(
          {
            error: 'Validation failed',
            details: error.errors,
            success: false,
          },
          { status: 400 }
        )
      }
      throw error
    }
  }
}
```

---

## Security Checklist (Both Stacks)

### Authentication
-  Verify authentication tokens
-  Handle missing/invalid tokens (401)
-  Check token expiration
-  Secure token storage

### Authorization
-  Check user roles/permissions (403)
-  Verify resource ownership
-  Implement least privilege principle
-  Log authorization failures

### Input Validation
-  Validate all inputs (Pydantic/Zod)
-  Sanitize SQL/NoSQL inputs
-  Escape special characters
-  Limit payload sizes

### Rate Limiting
-  Per-user limits
-  Per-IP limits
-  Clear error messages (429)
-  Retry-After headers

### Error Handling
-  Don't expose stack traces
-  Generic error messages to client
-  Log detailed errors server-side
-  Consistent error format

---

## What to Generate

1. **Protected Route Handler** - Secured version of the API route
2. **Auth Middleware/Utilities** - Reusable authentication helpers
3. **Type Definitions** - User, permissions, roles
4. **Error Responses** - Standardized auth errors
5. **Usage Examples** - Client-side integration

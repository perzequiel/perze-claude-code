---
description: Create a new API route with validation, error handling, and type safety
model: claude-sonnet-4-5
polyglot: true
---

Create a new API route following modern best practices.

## Requirements

API Endpoint: $ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Python Stack (py) - FastAPI

### File Structure

```
src/
├── api/
│   └── routes/
│       └── endpoint.py      # Route handler
├── schemas/
│   └── endpoint.py          # Pydantic models
├── services/
│   └── endpoint_service.py  # Business logic
└── main.py                  # FastAPI app
```

### Route Handler

```python
# src/api/routes/endpoint.py
from fastapi import APIRouter, Depends, HTTPException, status
from sqlalchemy.ext.asyncio import AsyncSession

from src.common.db.session import get_db
from src.schemas.endpoint import (
    EndpointCreate,
    EndpointResponse,
    EndpointUpdate,
)
from src.services.endpoint_service import EndpointService

router = APIRouter(prefix="/endpoint", tags=["endpoint"])


@router.post(
    "/",
    response_model=EndpointResponse,
    status_code=status.HTTP_201_CREATED,
)
async def create_endpoint(
    data: EndpointCreate,
    db: AsyncSession = Depends(get_db),
) -> EndpointResponse:
    """Create a new endpoint resource."""
    service = EndpointService(db)
    result = await service.create(data)
    return result


@router.get("/{endpoint_id}", response_model=EndpointResponse)
async def get_endpoint(
    endpoint_id: int,
    db: AsyncSession = Depends(get_db),
) -> EndpointResponse:
    """Get endpoint by ID."""
    service = EndpointService(db)
    result = await service.get_by_id(endpoint_id)
    if not result:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Endpoint {endpoint_id} not found",
        )
    return result


@router.put("/{endpoint_id}", response_model=EndpointResponse)
async def update_endpoint(
    endpoint_id: int,
    data: EndpointUpdate,
    db: AsyncSession = Depends(get_db),
) -> EndpointResponse:
    """Update endpoint by ID."""
    service = EndpointService(db)
    result = await service.update(endpoint_id, data)
    if not result:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Endpoint {endpoint_id} not found",
        )
    return result


@router.delete("/{endpoint_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_endpoint(
    endpoint_id: int,
    db: AsyncSession = Depends(get_db),
) -> None:
    """Delete endpoint by ID."""
    service = EndpointService(db)
    deleted = await service.delete(endpoint_id)
    if not deleted:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail=f"Endpoint {endpoint_id} not found",
        )
```

### Pydantic Schemas

```python
# src/schemas/endpoint.py
from pydantic import BaseModel, Field


class EndpointBase(BaseModel):
    """Base schema with shared fields."""

    name: str = Field(..., min_length=1, max_length=100)
    description: str | None = None


class EndpointCreate(EndpointBase):
    """Schema for creating endpoint."""

    pass


class EndpointUpdate(BaseModel):
    """Schema for updating endpoint (all fields optional)."""

    name: str | None = Field(None, min_length=1, max_length=100)
    description: str | None = None


class EndpointResponse(EndpointBase):
    """Schema for endpoint response."""

    id: int
    created_at: str
    updated_at: str

    model_config = {"from_attributes": True}
```

### Response Format

```python
# Success
{"id": 1, "name": "example", "created_at": "...", "updated_at": "..."}

# Error (FastAPI automatic)
{"detail": "Endpoint 123 not found"}

# Validation Error (FastAPI automatic)
{"detail": [{"loc": ["body", "name"], "msg": "field required", "type": "value_error.missing"}]}
```

---

## TypeScript Stack (ts) - Next.js App Router

### File Structure

```
app/
└── api/
    └── endpoint/
        ├── route.ts           # GET all, POST
        └── [id]/
            └── route.ts       # GET one, PUT, DELETE
lib/
├── validations/
│   └── endpoint.ts            # Zod schemas
└── types/
    └── endpoint.ts            # TypeScript types
```

### Route Handler

```typescript
// app/api/endpoint/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'

// Validation schema
const createEndpointSchema = z.object({
  name: z.string().min(1).max(100),
  description: z.string().optional(),
})

// GET /api/endpoint
export async function GET() {
  try {
    // Fetch data from database
    const data = await getEndpoints()

    return NextResponse.json({ data, success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch endpoints', success: false },
      { status: 500 }
    )
  }
}

// POST /api/endpoint
export async function POST(request: NextRequest) {
  try {
    const body = await request.json()

    // Validate input
    const validation = createEndpointSchema.safeParse(body)
    if (!validation.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: validation.error.issues, success: false },
        { status: 400 }
      )
    }

    // Create resource
    const result = await createEndpoint(validation.data)

    return NextResponse.json(
      { data: result, success: true },
      { status: 201 }
    )
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to create endpoint', success: false },
      { status: 500 }
    )
  }
}
```

```typescript
// app/api/endpoint/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'
import { z } from 'zod'

const updateEndpointSchema = z.object({
  name: z.string().min(1).max(100).optional(),
  description: z.string().optional(),
})

type Params = { params: Promise<{ id: string }> }

// GET /api/endpoint/[id]
export async function GET(request: NextRequest, { params }: Params) {
  try {
    const { id } = await params
    const data = await getEndpointById(id)

    if (!data) {
      return NextResponse.json(
        { error: 'Endpoint not found', success: false },
        { status: 404 }
      )
    }

    return NextResponse.json({ data, success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch endpoint', success: false },
      { status: 500 }
    )
  }
}

// PUT /api/endpoint/[id]
export async function PUT(request: NextRequest, { params }: Params) {
  try {
    const { id } = await params
    const body = await request.json()

    const validation = updateEndpointSchema.safeParse(body)
    if (!validation.success) {
      return NextResponse.json(
        { error: 'Validation failed', details: validation.error.issues, success: false },
        { status: 400 }
      )
    }

    const result = await updateEndpoint(id, validation.data)

    if (!result) {
      return NextResponse.json(
        { error: 'Endpoint not found', success: false },
        { status: 404 }
      )
    }

    return NextResponse.json({ data: result, success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to update endpoint', success: false },
      { status: 500 }
    )
  }
}

// DELETE /api/endpoint/[id]
export async function DELETE(request: NextRequest, { params }: Params) {
  try {
    const { id } = await params
    const deleted = await deleteEndpoint(id)

    if (!deleted) {
      return NextResponse.json(
        { error: 'Endpoint not found', success: false },
        { status: 404 }
      )
    }

    return new NextResponse(null, { status: 204 })
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to delete endpoint', success: false },
      { status: 500 }
    )
  }
}
```

### Response Format

```typescript
// Success
{ data: T, success: true }

// Error
{ error: string, details?: unknown, success: false }
```

---

## Best Practices (Both Stacks)

### Do
- Early validation before expensive operations
- Proper HTTP status codes (200, 201, 400, 401, 404, 500)
- Consistent error response format
- Strict typing (no `any` in TS, type hints in Python)
- Minimal logic in routes (use services/utils)
- Request/response logging for debugging

### Don't
- Expose sensitive data in responses
- Execute database queries without validation
- Put business logic inline in routes (extract to services)
- Use generic error messages

---

## What to Generate

1. **Route Handler File** - Complete route implementation
2. **Validation Schema** - Pydantic models or Zod schemas
3. **Type Definitions** - Shared types
4. **Error Handler** - Centralized error handling
5. **Example Usage** - Client-side fetch/httpx example

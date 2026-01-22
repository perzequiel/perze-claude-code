---
description: Test API endpoints with automated test generation
model: claude-sonnet-4-5
polyglot: true
---

Generate comprehensive API tests for the specified endpoint.

## Target

$ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Python Stack (py)

### Testing Tools

- **pytest** - Primary testing framework
- **pytest-asyncio** - Async test support
- **httpx** - Async HTTP client for API testing
- **pytest-cov** - Coverage reporting
- **respx** - HTTP mocking for httpx

### Run Tests

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=src --cov-report=html

# Run specific test file
pytest tests/test_api.py -v

# Run tests matching pattern
pytest -k "test_create" -v
```

### Test Structure

```python
# tests/test_api.py
import pytest
from httpx import AsyncClient, ASGITransport
from src.main import app


class TestAPIEndpoint:
    """Tests for API endpoint."""

    @pytest.fixture
    async def client(self):
        """Create async test client."""
        transport = ASGITransport(app=app)
        async with AsyncClient(transport=transport, base_url="http://test") as client:
            yield client

    # Success Cases
    async def test_should_handle_valid_request(self, client: AsyncClient):
        """Test valid request returns expected response."""
        response = await client.post(
            "/api/endpoint",
            json={"key": "value"}
        )
        assert response.status_code == 200
        assert response.json()["success"] is True

    async def test_should_return_correct_status_code(self, client: AsyncClient):
        """Test correct HTTP status codes."""
        response = await client.get("/api/endpoint/123")
        assert response.status_code == 200

    # Validation Tests
    async def test_should_reject_invalid_input(self, client: AsyncClient):
        """Test validation rejects invalid data."""
        response = await client.post(
            "/api/endpoint",
            json={"invalid": "data"}
        )
        assert response.status_code == 422

    async def test_should_validate_required_fields(self, client: AsyncClient):
        """Test required fields are enforced."""
        response = await client.post("/api/endpoint", json={})
        assert response.status_code == 422

    # Error Handling Tests
    async def test_should_handle_not_found(self, client: AsyncClient):
        """Test 404 for non-existent resources."""
        response = await client.get("/api/endpoint/nonexistent")
        assert response.status_code == 404

    async def test_should_return_proper_error_format(self, client: AsyncClient):
        """Test error response format."""
        response = await client.get("/api/endpoint/invalid")
        assert "error" in response.json()
```

### Python Configuration

**pyproject.toml**
```toml
[tool.pytest.ini_options]
testpaths = ["tests"]
asyncio_mode = "auto"
addopts = "-v --tb=short"

[tool.coverage.run]
source = ["src"]
omit = ["tests/*"]
```

---

## TypeScript Stack (ts)

### Testing Tools

- **Vitest** - Fast, modern (recommended for new projects)
- **Jest** - Established, widely used
- **Supertest** - HTTP assertions
- **MSW** - API mocking

### Run Tests

```bash
# Run all tests
npm test

# Run with coverage
npm test -- --coverage

# Run specific test file
npm test -- api.test.ts

# Watch mode
npm test -- --watch
```

### Test Structure

```typescript
// __tests__/api.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest'
import { createServer } from 'http'

describe('API Endpoint', () => {
  // Success Cases
  describe('Success Cases', () => {
    it('should handle valid request', async () => {
      const response = await fetch('/api/endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ key: 'value' })
      })

      expect(response.status).toBe(200)
      const data = await response.json()
      expect(data.success).toBe(true)
    })

    it('should return correct status code', async () => {
      const response = await fetch('/api/endpoint/123')
      expect(response.status).toBe(200)
    })
  })

  // Validation Tests
  describe('Validation', () => {
    it('should reject invalid input', async () => {
      const response = await fetch('/api/endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ invalid: 'data' })
      })

      expect(response.status).toBe(400)
    })

    it('should validate required fields', async () => {
      const response = await fetch('/api/endpoint', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({})
      })

      expect(response.status).toBe(400)
    })
  })

  // Error Handling Tests
  describe('Error Handling', () => {
    it('should handle not found', async () => {
      const response = await fetch('/api/endpoint/nonexistent')
      expect(response.status).toBe(404)
    })

    it('should return proper error format', async () => {
      const response = await fetch('/api/endpoint/invalid')
      const data = await response.json()
      expect(data).toHaveProperty('error')
    })
  })
})
```

### TypeScript Configuration

**vitest.config.ts**
```typescript
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    environment: 'node',
    include: ['**/*.test.ts'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'html']
    }
  }
})
```

---

## Test Coverage (Both Stacks)

### Happy Paths
- Valid inputs return expected results
- Proper status codes
- Correct response structure

### Error Paths
- Invalid input validation
- Authentication failures
- Rate limiting
- Server errors
- Missing required fields

### Edge Cases
- Empty requests
- Malformed JSON
- Large payloads
- Special characters
- SQL injection attempts
- XSS attempts

### Authentication/Authorization
- Valid tokens
- Expired tokens
- Missing tokens
- Invalid permissions

---

## Key Testing Principles

- Test behavior, not implementation
- Clear, descriptive test names
- Arrange-Act-Assert pattern
- Independent tests (no shared state)
- Fast execution (<5s for unit tests)
- Realistic mock data

---

## What to Generate

1. **Test File** - Complete test suite with all scenarios
2. **Mock Data** - Realistic test fixtures
3. **Helper Functions** - Reusable test utilities
4. **Setup/Teardown** - Database/state management
5. **Run Script** - `pytest` or `npm test` ready

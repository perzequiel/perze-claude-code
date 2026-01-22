---
description: Refactor and clean up code following best practices
model: claude-sonnet-4-5
polyglot: true
---

Clean up and refactor the following code to improve readability, maintainability, and follow best practices.

## Code to Clean

$ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Code Smells to Fix (Both Stacks)

### Naming
-  Descriptive variable/function names
-  Consistent naming conventions
-  Avoid abbreviations unless obvious
-  Boolean names start with is/has/can

### Functions
-  Single responsibility per function
-  Keep functions small (<50 lines)
-  Reduce parameters (max 3-4)
-  Extract complex logic
-  Avoid side effects where possible

### DRY (Don't Repeat Yourself)
-  Extract repeated code to utilities
-  Create reusable components/functions
-  Centralize constants/configuration

### Complexity
-  Reduce nested if statements
-  Replace complex conditions with functions
-  Use early returns
-  Simplify boolean logic

---

## Python Stack (py)

### Python Naming Conventions

```python
# Variables and functions: snake_case
user_name = "john"
def get_user_by_id(user_id: int) -> User:
    pass

# Classes: PascalCase
class UserService:
    pass

# Constants: UPPER_SNAKE_CASE
MAX_RETRIES = 3
DEFAULT_TIMEOUT = 30

# Private: leading underscore
class Service:
    def _internal_method(self):
        pass
```

### Python Type Hints

```python
# Before: No type hints
def process_data(data, options=None):
    result = []
    for item in data:
        if item.get('active'):
            result.append(item)
    return result

# After: Full type hints
from typing import TypedDict

class DataItem(TypedDict):
    id: int
    active: bool
    value: str

class ProcessOptions(TypedDict, total=False):
    include_inactive: bool

def process_data(
    data: list[DataItem],
    options: ProcessOptions | None = None,
) -> list[DataItem]:
    options = options or {}
    include_inactive = options.get("include_inactive", False)

    return [
        item for item in data
        if item["active"] or include_inactive
    ]
```

### Python Refactoring Patterns

**Extract function**
```python
# Before: Long function
async def create_user(data: UserCreate, db: AsyncSession) -> User:
    # Validation (20 lines)
    if not data.email:
        raise ValueError("Email required")
    if not re.match(EMAIL_PATTERN, data.email):
        raise ValueError("Invalid email")
    existing = await db.execute(select(User).where(User.email == data.email))
    if existing.scalar():
        raise ValueError("Email exists")

    # Creation (10 lines)
    user = User(**data.model_dump())
    db.add(user)
    await db.commit()
    return user

# After: Extracted functions
async def create_user(data: UserCreate, db: AsyncSession) -> User:
    await _validate_user_data(data, db)
    return await _save_user(data, db)

async def _validate_user_data(data: UserCreate, db: AsyncSession) -> None:
    if not data.email:
        raise ValueError("Email required")
    if not re.match(EMAIL_PATTERN, data.email):
        raise ValueError("Invalid email")
    if await _email_exists(data.email, db):
        raise ValueError("Email exists")

async def _email_exists(email: str, db: AsyncSession) -> bool:
    result = await db.execute(select(User).where(User.email == email))
    return result.scalar() is not None

async def _save_user(data: UserCreate, db: AsyncSession) -> User:
    user = User(**data.model_dump())
    db.add(user)
    await db.commit()
    return user
```

**Replace conditionals with mapping**
```python
# Before: Multiple if statements
def get_status_message(status: str) -> str:
    if status == "pending":
        return "Waiting for approval"
    elif status == "approved":
        return "Request approved"
    elif status == "rejected":
        return "Request rejected"
    else:
        return "Unknown status"

# After: Dictionary mapping
STATUS_MESSAGES: dict[str, str] = {
    "pending": "Waiting for approval",
    "approved": "Request approved",
    "rejected": "Request rejected",
}

def get_status_message(status: str) -> str:
    return STATUS_MESSAGES.get(status, "Unknown status")
```

**Use Pydantic for data structures**
```python
# Before: Raw dictionaries
def create_response(user, token):
    return {
        "user": {
            "id": user.id,
            "email": user.email,
        },
        "token": token,
        "expires_at": datetime.now() + timedelta(hours=24),
    }

# After: Pydantic models
class UserResponse(BaseModel):
    id: int
    email: str

class AuthResponse(BaseModel):
    user: UserResponse
    token: str
    expires_at: datetime

def create_response(user: User, token: str) -> AuthResponse:
    return AuthResponse(
        user=UserResponse(id=user.id, email=user.email),
        token=token,
        expires_at=datetime.now() + timedelta(hours=24),
    )
```

---

## TypeScript Stack (ts)

### TypeScript Naming Conventions

```typescript
// Variables and functions: camelCase
const userName = 'john'
function getUserById(userId: string): User {}

// Classes and types: PascalCase
class UserService {}
interface UserData {}
type UserId = string

// Constants: UPPER_SNAKE_CASE or camelCase
const MAX_RETRIES = 3
const defaultTimeout = 30

// React components: PascalCase
function UserCard({ user }: UserCardProps) {}
```

### TypeScript Type Improvements

```typescript
// Before: Using any
function process(data: any, options?: any) {
  return data.map((item: any) => item.value)
}

// After: Proper types
interface DataItem {
  id: number
  value: string
  active: boolean
}

interface ProcessOptions {
  includeInactive?: boolean
}

function process(
  data: DataItem[],
  options?: ProcessOptions
): string[] {
  const { includeInactive = false } = options ?? {}
  return data
    .filter(item => item.active || includeInactive)
    .map(item => item.value)
}
```

### TypeScript Refactoring Patterns

**Extract function**
```typescript
// Before: Long function
async function createUser(data: UserCreate): Promise<User> {
  // Validation
  if (!data.email) throw new Error('Email required')
  if (!EMAIL_REGEX.test(data.email)) throw new Error('Invalid email')
  const existing = await db.user.findUnique({ where: { email: data.email } })
  if (existing) throw new Error('Email exists')

  // Creation
  const user = await db.user.create({ data })
  return user
}

// After: Extracted functions
async function createUser(data: UserCreate): Promise<User> {
  await validateUserData(data)
  return saveUser(data)
}

async function validateUserData(data: UserCreate): Promise<void> {
  if (!data.email) throw new Error('Email required')
  if (!EMAIL_REGEX.test(data.email)) throw new Error('Invalid email')
  if (await emailExists(data.email)) throw new Error('Email exists')
}

async function emailExists(email: string): Promise<boolean> {
  const user = await db.user.findUnique({ where: { email } })
  return user !== null
}

async function saveUser(data: UserCreate): Promise<User> {
  return db.user.create({ data })
}
```

**Use modern JavaScript patterns**
```typescript
// Before
const value = obj && obj.prop && obj.prop.nested
const result = value !== null && value !== undefined ? value : defaultValue
const { name, email } = user ? user : {}

// After
const value = obj?.prop?.nested
const result = value ?? defaultValue
const { name, email } = user ?? {}
```

**React: Extract custom hooks**
```typescript
// Before: Logic in component
function UserProfile({ userId }: Props) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    setLoading(true)
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [userId])

  if (loading) return <Spinner />
  if (error) return <Error message={error.message} />
  if (!user) return null

  return <Profile user={user} />
}

// After: Custom hook
function useUser(userId: string) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<Error | null>(null)

  useEffect(() => {
    setLoading(true)
    fetchUser(userId)
      .then(setUser)
      .catch(setError)
      .finally(() => setLoading(false))
  }, [userId])

  return { user, loading, error }
}

function UserProfile({ userId }: Props) {
  const { user, loading, error } = useUser(userId)

  if (loading) return <Spinner />
  if (error) return <Error message={error.message} />
  if (!user) return null

  return <Profile user={user} />
}
```

**Replace conditionals with object mapping**
```typescript
// Before
function getStatusMessage(status: string): string {
  if (status === 'pending') return 'Waiting for approval'
  if (status === 'approved') return 'Request approved'
  if (status === 'rejected') return 'Request rejected'
  return 'Unknown status'
}

// After
const STATUS_MESSAGES: Record<string, string> = {
  pending: 'Waiting for approval',
  approved: 'Request approved',
  rejected: 'Request rejected',
}

function getStatusMessage(status: string): string {
  return STATUS_MESSAGES[status] ?? 'Unknown status'
}
```

---

## Common Cleanup Tasks (Both Stacks)

### Remove Dead Code
- Unused imports
- Unreachable code
- Commented out code
- Unused variables

### Improve Error Handling
- Catch specific exceptions
- Provide meaningful error messages
- Log errors with context
- Don't swallow errors silently

### Better Comments
- Remove obvious comments
- Add why, not what
- Document complex logic
- Update outdated comments

---

## Output Format

1. **Issues Found** - List of code smells and problems
2. **Cleaned Code** - Refactored version
3. **Explanations** - What changed and why
4. **Before/After Comparison** - Side-by-side if helpful
5. **Further Improvements** - Optional enhancements

Focus on practical improvements that make code more maintainable without over-engineering. Balance clean code with pragmatism.

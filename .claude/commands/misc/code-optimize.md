---
description: Analyze and optimize code for performance, memory, and efficiency
model: claude-sonnet-4-5
polyglot: true
---

Optimize the following code for performance and efficiency.

## Code to Optimize

$ARGUMENTS

## Stack Detection

First, detect the project stack:
1. Check for `py` or `ts` prefix in the command
2. If no prefix, auto-detect based on project files:
   - **Python**: `pyproject.toml`, `requirements.txt`
   - **TypeScript**: `package.json`, `tsconfig.json`

---

## Optimization Strategy

### 1. **Profiling First**
- Identify actual bottlenecks
- Don't optimize prematurely
- Measure before and after
- Focus on high-impact areas

---

## Python Stack (py)

### Performance Optimization Areas

**Async Operations**
- Use `asyncio.gather()` for concurrent I/O
- Avoid blocking calls in async functions
- Use `aiohttp`/`httpx` for async HTTP
- Connection pooling for databases

**Database Queries**
- Use async SQLAlchemy with connection pooling
- Add indexes for frequently queried fields
- Batch queries (reduce N+1 problems)
- Use `select()` to limit fields
- Implement pagination

**Memory Optimization**
- Use generators for large datasets
- Use `__slots__` for memory-heavy classes
- Avoid unnecessary object copies
- Use itertools for efficient iteration

**CPU Optimization**
- Use list comprehensions over loops
- Use `functools.lru_cache` for memoization
- Use `collections.defaultdict` and `Counter`
- Consider `numpy` for numerical operations

### Python Optimization Examples

**Replace inefficient patterns**
```python
# Before: Multiple iterations
result = []
for item in items:
    if item.active:
        result.append(item.value * 2)
total = sum(result)

# After: Generator expression
total = sum(item.value * 2 for item in items if item.active)
```

**Async concurrent operations**
```python
# Before: Sequential (slow)
async def fetch_all(urls: list[str]) -> list[dict]:
    results = []
    for url in urls:
        result = await fetch_url(url)
        results.append(result)
    return results

# After: Concurrent (fast)
async def fetch_all(urls: list[str]) -> list[dict]:
    tasks = [fetch_url(url) for url in urls]
    return await asyncio.gather(*tasks)
```

**Memoization with cache**
```python
from functools import lru_cache

# Before: Recalculates every time
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)

# After: Cached results
@lru_cache(maxsize=128)
def fibonacci(n: int) -> int:
    if n < 2:
        return n
    return fibonacci(n - 1) + fibonacci(n - 2)
```

**Database batch operations**
```python
# Before: N+1 queries
async def get_users_with_orders(db: AsyncSession) -> list[User]:
    users = await db.execute(select(User))
    for user in users.scalars():
        orders = await db.execute(
            select(Order).where(Order.user_id == user.id)
        )
        user.orders = orders.scalars().all()
    return users

# After: Eager loading
async def get_users_with_orders(db: AsyncSession) -> list[User]:
    query = select(User).options(selectinload(User.orders))
    result = await db.execute(query)
    return result.scalars().all()
```

### Python Measurement Tools

```bash
# CPU profiling
python -m cProfile -o profile.stats script.py
python -m pstats profile.stats

# Memory profiling
pip install memory-profiler
python -m memory_profiler script.py

# Line-by-line profiling
pip install line-profiler
kernprof -l -v script.py

# Async profiling
pip install aiomonitor
```

---

## TypeScript Stack (ts)

### Performance Optimization Areas

**React/Next.js**
- Memoization (React.memo, useMemo, useCallback)
- Code splitting and lazy loading
- Image optimization (next/image)
- Remove unnecessary re-renders
- Virtual scrolling for long lists

**Database Queries**
- Add indexes for frequently queried fields
- Batch queries (reduce N+1 problems)
- Use select to limit fields
- Implement pagination
- Use database views for complex joins

**API Calls**
- Implement caching (SWR, React Query)
- Debounce/throttle requests
- Parallel requests where possible
- Request deduplication
- Optimistic updates

**Bundle Size**
- Tree-shaking unused code
- Dynamic imports for large libraries
- Replace heavy dependencies
- Code splitting by route

### TypeScript Optimization Examples

**Memoize expensive calculations**
```typescript
// Before: Recalculates on every render
function Component({ data }: Props) {
  const processedData = expensiveCalculation(data)
  return <List items={processedData} />
}

// After: Memoized
function Component({ data }: Props) {
  const processedData = useMemo(
    () => expensiveCalculation(data),
    [data]
  )
  return <List items={processedData} />
}
```

**Prevent unnecessary re-renders**
```typescript
// Before: New function reference every render
function Parent() {
  return <Child onClick={() => handleClick()} />
}

// After: Stable reference
function Parent() {
  const handleChildClick = useCallback(() => {
    handleClick()
  }, [])
  return <Child onClick={handleChildClick} />
}
```

**Dynamic imports for code splitting**
```typescript
// Before: Loads immediately
import HeavyComponent from './HeavyComponent'

// After: Loads on demand
const HeavyComponent = dynamic(() => import('./HeavyComponent'), {
  loading: () => <Spinner />,
})
```

**Replace inefficient array methods**
```typescript
// Before: Multiple iterations
const result = arr
  .filter(x => x > 0)
  .map(x => x * 2)
  .reduce((sum, x) => sum + x, 0)

// After: Single iteration
const result = arr.reduce((sum, x) => {
  return x > 0 ? sum + (x * 2) : sum
}, 0)
```

**Virtual scrolling for long lists**
```typescript
// Before: Renders all items
function List({ items }: { items: Item[] }) {
  return (
    <div>
      {items.map(item => <ListItem key={item.id} item={item} />)}
    </div>
  )
}

// After: Only renders visible items
import { useVirtualizer } from '@tanstack/react-virtual'

function List({ items }: { items: Item[] }) {
  const parentRef = useRef<HTMLDivElement>(null)
  const virtualizer = useVirtualizer({
    count: items.length,
    getScrollElement: () => parentRef.current,
    estimateSize: () => 50,
  })

  return (
    <div ref={parentRef} style={{ overflow: 'auto', height: '400px' }}>
      <div style={{ height: virtualizer.getTotalSize() }}>
        {virtualizer.getVirtualItems().map(virtualRow => (
          <ListItem key={virtualRow.key} item={items[virtualRow.index]} />
        ))}
      </div>
    </div>
  )
}
```

### TypeScript Measurement Tools

```bash
# Bundle analysis
npm install @next/bundle-analyzer
ANALYZE=true npm run build

# React DevTools Profiler
# Use browser extension

# Lighthouse CI
npm install -g lighthouse
lighthouse http://localhost:3000 --view

# Load testing
npm install -g k6
k6 run loadtest.js
```

---

## Optimization Checklist (Both Stacks)

### Database
-  Add indexes on frequently queried columns
-  Use prepared statements / parameterized queries
-  Batch inserts/updates
-  Implement connection pooling
-  Cache expensive queries

### Network
-  Compress responses (gzip/brotli)
-  Use CDN for static assets
-  Set proper cache headers
-  Minimize payload size
-  Use HTTP/2

### Memory
-  Fix memory leaks
-  Avoid unnecessary object creation
-  Use streaming for large data
-  Clear timers and listeners
-  Use appropriate data structures

---

## Output Format

1. **Analysis** - Identify performance bottlenecks
2. **Optimized Code** - Improved version
3. **Explanation** - What changed and why
4. **Benchmarks** - Expected performance improvement
5. **Trade-offs** - Any complexity added
6. **Next Steps** - Further optimization opportunities

Focus on practical, measurable optimizations that provide real user value. Don't sacrifice readability for micro-optimizations.

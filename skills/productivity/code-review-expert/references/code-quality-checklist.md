# Code Quality Checklist

## Error Handling

### Anti-patterns to Flag

- **Swallowed exceptions**: Empty catch blocks or catch that only logs

```python
# ❌ Silent failure
try:
    ...
except Exception:
    pass

# ❌ Log and forget — caller doesn't know it failed
try:
    ...
except Exception as e:
    logging.error(e)
```

- **Overly broad catch**: Catching `Exception`/`BaseException` instead of specific types
- **Error information leakage**: Stack traces or internal details exposed to users
- **Missing error handling**: No try-except around fallible operations (I/O, network, parsing)
- **Async error handling**: Unhandled promise rejections, missing `.catch()`, no error boundary

### Best Practices to Check

- [ ] Errors are caught at appropriate boundaries
- [ ] Error messages are user-friendly (no internal details exposed)
- [ ] Errors are logged with sufficient context for debugging
- [ ] Async errors are properly propagated or handled
- [ ] Fallback behavior is defined for recoverable errors
- [ ] Critical errors trigger alerts/monitoring

### Questions to Ask

- "What happens when this operation fails?"
- "Will the caller know something went wrong?"
- "Is there enough context to debug this error?"

---

## Performance & Caching

### CPU-Intensive Operations

- **Expensive operations in hot paths**: Regex compilation, JSON parsing, crypto in loops
- **Blocking main thread**: Sync I/O, heavy computation without worker/async
- **Unnecessary recomputation**: Same calculation done multiple times
- **Missing memoization**: Pure functions called repeatedly with same inputs

### Database & I/O

- **N+1 queries**: Loop that makes a query per item instead of batch

```python
# ❌ N+1
for id in ids:
    user = db.query("SELECT * FROM users WHERE id = ?", id)

# ✅ Batch
users = db.query("SELECT * FROM users WHERE id IN (?)", ids)
```

- **Missing indexes**: Queries on unindexed columns
- **Over-fetching**: `SELECT *` when only few columns needed
- **No pagination**: Loading entire dataset into memory

### Caching Issues

- **Missing cache for expensive operations**: Repeated API calls, DB queries, computations
- **Cache without TTL**: Stale data served indefinitely
- **Cache without invalidation strategy**: Data updated but cache not cleared
- **Cache key collisions**: Insufficient key uniqueness
- **Caching user-specific data globally**: Security/privacy issue

### Memory

- **Unbounded collections**: Arrays/maps that grow without limit
- **Large object retention**: Holding references preventing GC
- **String concatenation in loops**: Use `''.join()` or `io.StringIO` instead
- **Loading large files entirely**: Use streaming/chunked reads instead

### Questions to Ask

- "What's the time complexity of this operation?"
- "How does this behave with 10x/100x data?"
- "Is this result cacheable? Should it be?"
- "Can this be batched instead of one-by-one?"

---

## Boundary Conditions

### Null/None Handling

- **Missing None checks**: Accessing attributes on potentially None objects
- **Truthy/falsy confusion**: `if value` when `0`, `""`, `[]`, or `False` are valid
- **Optional chaining overuse**: Hiding structural issues behind deep attribute access chains
- **None vs sentinel inconsistency**: Mixed usage without clear convention

### Empty Collections

- **Empty sequence not handled**: Code assumes iterable has items
- **Empty dict edge case**: `for key in dict` or `.items()` on empty dict
- **First/last element access**: `seq[0]` or `seq[-1]` without length check

### Numeric Boundaries

- **Division by zero**: Missing check before division
- **Integer overflow**: Large numbers exceeding safe integer range (Python ints are unbounded, but floats/decimal are not)
- **Floating point comparison**: Using `==` instead of tolerance-based comparison
- **Negative values**: Index or count that shouldn't be negative
- **Off-by-one errors**: Loop bounds, array slicing, pagination

### String Boundaries

- **Empty string**: Not handled as edge case
- **Whitespace-only string**: Passes truthy check but is effectively empty
- **Very long strings**: No length limits causing memory/display issues
- **Unicode edge cases**: Emoji, RTL text, combining characters

### Common Patterns to Flag

```python
# ❌ No None check
name = user.profile.name  # AttributeError if profile is None

# ❌ Sequence access without check
first = items[0]  # IndexError if items is empty

# ❌ Division without check
avg = total / count  # ZeroDivisionError if count is 0

# ❌ Truthy check excludes valid values
if value:  # fails for 0, "", [], False
    ...
```

### Questions to Ask

- "What if this is None?"
- "What if this collection is empty?"
- "What's the valid range for this number?"
- "What happens at the boundaries (0, -1, MAX_INT)?"

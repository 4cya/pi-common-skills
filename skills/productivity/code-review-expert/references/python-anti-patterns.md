# Python Anti-Patterns Checklist

Language-specific pitfalls that should be flagged during Python code review.

## Mutable Default Arguments (Classic Trap)

Default arguments are evaluated **once at function definition**, not each call.

```python
# ❌ P1: Shared mutable state across calls
def add_item(item, items=[]):
    items.append(item)
    return items

# ✅ Safe: None sentinel
def add_item(item, items=None):
    if items is None:
        items = []
    items.append(item)
    return items
```

## Late Binding Closures

Loop variables are captured by reference, not value.

```python
# ❌ P1: All lambdas return the same final value
funcs = [lambda: i for i in range(5)]  # All return 4

# ✅ Freeze with default argument
funcs = [lambda i=i: i for i in range(5)]
```

## Context Manager Neglect

Always use `with` for resources that need cleanup.

```python
# ❌ P1: Resource leak if exception occurs
f = open('file.txt')
data = f.read()
f.close()  # May not execute

# ✅ Context manager guarantees cleanup
with open('file.txt') as f:
    data = f.read()
```

Also applies to: `threading.Lock`, `socket.socket`, `tempfile`, custom `__enter__/__exit__` objects.

## Type Annotation Gaps (PEP 484)

- Public functions missing type annotations entirely
- Overusing `Any` — defeats the purpose of static type checking
- Using `Optional[X]` incorrectly (note: `Optional[X]` = `Union[X, None]`)
- Missing return type annotation on non-trivial functions
- Incorrect generic types (e.g., `list` vs `list[str]`)

```python
# ❌ P2: Missing/weak type annotations
def process(data, config):
    return data.get('result')

# ✅ Clear types
def process(data: dict[str, Any], config: Config) -> Optional[str]:
    return data.get('result')
```

## PEP 8 Violations

- Line length > 120 characters
- Naming: `snake_case` functions/vars, `PascalCase` classes, `UPPER_CASE` constants
- Whitespace: 2 blank lines between top-level functions, 1 between class methods, space after comma
- Import order: stdlib → third-party → local (alphabetical within groups)
- Unused imports

## asyncio Pitfalls

- **Sync blocking in async**: `time.sleep()` inside `async def` → blocks event loop. Use `await asyncio.sleep()`.
- **Forgotten `await`**: Coroutine object created but never awaited → never executes.
- **CPU-bound work on event loop**: Heavy computation blocks all async tasks. Use `loop.run_in_executor()` or `multiprocessing`.
- **Lost task references**: `asyncio.create_task()` result not stored → task may be GC'd before completion.
- **Unbounded queues**: Producer-consumer pattern without `asyncio.Queue(maxsize=N)`.

```python
# ❌ P1: Blocks entire event loop
async def handler():
    time.sleep(5)  # Sync sleep! Blocks all coroutines.
    result = heavy_cpu_work()  # Blocks event loop.
    task = asyncio.create_task(work())  # Reference lost, task may be GC'd

# ✅ Correct
async def handler():
    await asyncio.sleep(5)
    result = await loop.run_in_executor(None, heavy_cpu_work)
    self.task = asyncio.create_task(work())  # Reference stored
```

## GIL Awareness

- **CPU-bound work on main thread** → blocks all other Python threads. Use `multiprocessing` instead of `threading`.
- **I/O-bound work** → `threading` or `asyncio` are fine (GIL released during I/O).
- **C extensions** can release the GIL for CPU-bound work.

## `@property` Abuse

```python
# ❌ P2: Property with side effects or heavy computation
class User:
    @property
    def orders(self):
        return db.query("SELECT * FROM orders WHERE user_id = ?", self.id)

# ✅ Heavy/side-effecting work belongs in methods
class User:
    def load_orders(self):
        return db.query("SELECT * FROM orders WHERE user_id = ?", self.id)
```

## Class Attribute vs Instance Attribute Confusion

```python
# ❌ P1: Mutable class attribute shared across all instances!
class User:
    permissions = []  # Shared by ALL instances!

    def add_permission(self, perm):
        self.permissions.append(perm)  # Appends to class attribute!

# ✅ Set mutable defaults in __init__
class User:
    def __init__(self):
        self.permissions = []
```

## Incorrect `super()` Usage

- `super().__init__()` not called in subclass `__init__` → parent state not initialized
- MRO (Method Resolution Order) confusion with multiple inheritance and `super()`
- Python 3: use `super()` without args. Python 2-style `super(ClassName, self)` is obsolete.

## Catching Too Broadly

```python
# ❌ P1: Catching BaseException catches KeyboardInterrupt, SystemExit
try:
    ...
except BaseException:
    ...

# ❌ P2: Catching Exception is still too broad in many contexts
try:
    ...
except Exception:
    ...

# ✅ Catch specific exceptions
try:
    ...
except (ValueError, KeyError) as e:
    ...
```

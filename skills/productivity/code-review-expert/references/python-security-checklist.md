# Python-Specific Security Checklist

This checklist covers security vulnerabilities unique or particularly dangerous in Python.
Always run these checks in addition to the general security checklist.

## Deserialization Attacks

All of these can lead to **arbitrary code execution** — always P0:

- **`pickle.loads()` / `cPickle.loads()`** — Never use on untrusted data. The pickle VM can execute arbitrary Python code during deserialization.
- **`yaml.load()`** — Uses the unsafe Loader by default. Always use `yaml.safe_load()` or explicit `yaml.Loader`.
- **`marshal.loads()`** — Internal Python serialization; no safety guarantees.
- **`shelve` module** — Backed by pickle; same risks.
- **Safe alternatives**: `json.loads()`, `yaml.safe_load()`, or `ast.literal_eval()` for simple literals only.

```python
# ❌ P0: Arbitrary code execution
data = pickle.loads(user_input)
config = yaml.load(user_config)

# ✅ Safe
data = json.loads(user_input)
config = yaml.safe_load(user_config)
```

## Code Execution

- **`eval()`** — Accepts arbitrary Python expressions. User input → P0.
- **`exec()`** — Accepts arbitrary Python statements. User input → P0.
- **`compile()`** — Compiles code for later execution. User input → P0.
- **`__import__()`** — Dynamic import with user-controlled string → P0.
- **`getattr(obj, user_controlled_attr)`** — Can access internal attributes (`__class__`, `__subclasses__`, etc.).
- **`setattr(obj, user_controlled_attr, value)`** — Can overwrite critical attributes.

```python
# ❌ P0: All of these are dangerous with user input
result = eval(user_input)
exec(user_input)
mod = __import__(user_input)
attr = getattr(obj, user_input)
```

## Command Injection (subprocess)

- **`subprocess.Popen(shell=True)`** / `subprocess.run(shell=True)` — User input in shell string → command injection. Always use `shell=False` + list args.
- **`os.system()`** / `os.popen()` — Passes string to shell. Prefer `subprocess.run()`.
- **`os.exec*()` family** — Same shell risks.

```python
# ❌ P1/P0: Command injection via shell=True
subprocess.run(f"ls {user_path}", shell=True)
os.system(f"ping {user_host}")

# ✅ Safe: shell=False + list args
subprocess.run(["ls", user_path], shell=False)
subprocess.run(["ping", "-c", "1", user_host], shell=False)
```

## Template Injection (SSTI)

- **Jinja2**: `render_template_string()` with user input → SSTI. User controls template content.
- **Django**: `mark_safe()` misuse in custom template tags.
- **Mako**: `Template(user_input).render()` without sandbox.
- Never concatenate user input into template source strings.

```python
# ❌ P0: SSTI - user controls template
jinja2.Template(user_input).render()
flask.render_template_string(user_input)

# ✅ Safe: user data goes to template variables, not template source
flask.render_template('page.html', user_name=user_input)
```

## Path Traversal (Python-specific quirks)

- **`os.path.join("/safe", "/etc/passwd")`** → `/etc/passwd` — The second absolute path **replaces** the first!
- Always validate normalized paths stay within the allowed directory.
- Use `pathlib.Path` and check `.resolve()` against allowed roots.

```python
# ❌ Python quirk: second absolute arg replaces first
safe_path = os.path.join("/var/uploads", filename)  # filename = "/etc/passwd" → "/etc/passwd"!

# ✅ Safe with pathlib
from pathlib import Path
base = Path("/var/uploads").resolve()
target = (base / filename).resolve()
if not str(target).startswith(str(base)):
    raise ValueError("Path traversal detected")
```

## Dependency Security

- Check `requirements.txt` / `pyproject.toml` for packages with known CVEs.
- Unpinned versions (`>=` without `==`) allow supply chain attacks.
- **Tools**: `pip-audit`, `safety check`, `pipenv check`.

## Logging Safety

- Avoid interpolating user input directly into log format strings (may leak sensitive data to log aggregators, SIEMs).
- Tracebacks should never be returned to API clients.

```python
# ❌ Sensitive data in logs
logging.info(f"User {password} login attempt")  # password logged!

# ✅ Redact sensitive fields
logging.info("User %s login attempt", user_id)
```

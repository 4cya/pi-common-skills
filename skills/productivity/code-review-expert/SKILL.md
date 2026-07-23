---
name: code-review-expert
description: >
  Expert Python code review of current git changes with a senior engineer lens.
  Detects SOLID violations, security risks (including Python-specific:
  pickle/eval/subprocess/SSTI), performance issues, error handling gaps,
  boundary conditions, and PEP8 compliance. P0-P3 severity grading.
  Review-first workflow — no changes until user confirms.
  Use when user says "review", "code review", "check my code",
  "review this PR", "/code-review-expert", or similar code review requests.
---

# Python Code Review Expert

Integrated from:
- [sanyuan0704/code-review-expert](https://github.com/sanyuan0704/sanyuan-skills) — workflow skeleton
- [yennanliu/python-code-reviewer](https://github.com/yennanliu/ai_experiment) — Python adaptation
- Custom Python-specific security and anti-pattern additions

## Severity Levels

| Level | Name | Description | Action |
|-------|------|-------------|--------|
| **P0** | Critical | Security vulnerability, data loss risk, correctness bug | Must block merge |
| **P1** | High | Logic error, significant SOLID violation, performance regression | Should fix before merge |
| **P2** | Medium | Code smell, maintainability concern, minor SOLID violation | Fix in this PR or create follow-up |
| **P3** | Low | Style, naming, PEP8, minor suggestion | Optional improvement |

## Workflow

### 1) Preflight — Scope Changes

- Run `git status -sb`, `git diff --stat`, and `git diff` to scope changes.
- Use `grep` or `rg` to find related modules, usages, and contracts.
- Identify entry points, auth paths, payment logic, and data write paths.

**Edge cases:**
- **No changes**: If `git diff` is empty, inform user and ask if they want to review staged changes or a specific commit range.
- **Large diff (>500 lines)**: Summarize by file first, then review in batches by module/feature area.
- **Mixed concerns**: Group findings by logical feature, not just file order.

### 2) SOLID + Architecture Smells

**Load `references/solid-checklist.md` for detailed prompts.**

Check for:
- **SRP**: Overloaded modules with unrelated responsibilities. Ask: "What is the single reason this module would change?"
- **OCP**: Frequent edits to add behavior instead of extension points. Ask: "Can I add a new variant without touching existing code?"
- **LSP**: Subclasses that break expectations or require type checks. Ask: "Can I substitute any subclass without the caller knowing?"
- **ISP**: Wide interfaces with unused methods. Ask: "Do all implementers use all methods?"
- **DIP**: High-level logic tied to low-level implementations. Ask: "Can I swap the implementation without changing business logic?"

When proposing a refactor, explain _why_ it improves cohesion/coupling and outline a minimal, safe split. If non-trivial, propose an incremental plan.

### 3) Removal Candidates + Iteration Plan

**Load `references/removal-plan.md` for template.**

- Identify code that is unused, redundant, or feature-flagged off.
- Distinguish **safe delete now** vs **defer with plan**.
- Run pre-removal checklist: search references, check dynamic usage, verify no external consumers, review telemetry, update tests and docs.

### 4) Security and Reliability Scan

**Load `references/security-checklist.md` for general coverage.**
**Load `references/python-security-checklist.md` for Python-specific checks.**

General checks:
- Injection (SQL/NoSQL/command/GraphQL), SSRF, path traversal
- AuthN/AuthZ gaps, IDOR, JWT algorithm confusion, missing tenancy checks
- Secret leakage, weak crypto, hardcoded credentials
- Race conditions (TOCTOU, concurrent state, DB concurrency, distributed locks)
- Data integrity (missing transactions, no idempotency, lost updates)
- Runtime risks (unbounded loops, missing timeouts, resource exhaustion)

Python-specific checks:
- **Deserialization**: `pickle.loads()`, `yaml.load()`, `marshal.loads()` → P0
- **Code execution**: `eval()`, `exec()`, `compile()` with user input → P0
- **Command injection**: `subprocess(shell=True)`, `os.system()` → P0/P1
- **SSTI**: Jinja2 `render_template_string()`, Django `mark_safe()` misuse → P0
- **Path traversal**: `os.path.join()` absolute-path quirk
- **Dependency CVEs**: unpinned versions in `requirements.txt`/`pyproject.toml`

### 5) Code Quality Scan

**Load `references/code-quality-checklist.md` for error handling, performance, and boundary conditions.**
**Load `references/python-anti-patterns.md` for Python-specific anti-patterns.**

General checks:
- **Error handling**: swallowed exceptions, overly broad catch, missing async error propagation
- **Performance**: N+1 queries, missing indexes, no pagination, unbounded memory, missing caching
- **Boundary conditions**: None/null handling, empty collections, numeric boundaries, off-by-one

Python-specific checks:
- **Mutable default arguments** (`def f(x=[])`) → P1
- **Late binding closures** (loop variable capture) → P1
- **Context manager neglect** (missing `with` for resources) → P1
- **Type annotations** (PEP 484 gaps, `Any` overuse) → P2
- **PEP 8 violations** (naming, whitespace, import order, line length) → P3
- **asyncio pitfalls** (sync blocking, forgotten await, lost task refs) → P1
- **GIL awareness** (CPU-bound in main thread) → P2
- **Class attribute confusion** (mutable class attrs vs instance attrs) → P1
- **Overly broad exception catching** → P1/P2

### 6) Output Format

Structure your review as follows:

```
## Code Review Summary

**Files reviewed**: X files, Y lines changed
**Overall assessment**: [APPROVE / REQUEST_CHANGES / COMMENT]

---

## ✅ Strengths

1. [What's working well]
2. [Good patterns observed]
...

---

## 🔴 Findings

### P0 - Critical
(none or list)

### P1 - High
1. **`file:line`** — Brief title
   - Description of issue
   - Suggested fix with Python code example
   ```python
   # Fix example
   ```

### P2 - Medium
2. (continue numbering across sections)
   - ...

### P3 - Low
...

---

## 🗑️ Removal/Iteration Plan
(if applicable)

## 📊 Overall Rating: X/10

**Summary:** [1-2 sentence summary of code quality and main concerns]

---

## 🛠️ Recommended Tool Checks

- `ruff check .` — Python linting & formatting
- `bandit -r .` — Python security scanning
- `mypy .` — Static type checking
- `pip-audit` — Dependency vulnerability scan
```

For file-specific inline comments:

```
::code-comment{file="path/to/file.py" line="42" severity="P1"}
Description of the issue and suggested fix (in Python).
::
```

**Clean review**: If no issues found, explicitly state:
- What was checked
- Any areas not covered (e.g., "Did not verify database migrations")
- Residual risks or recommended follow-up tests

### 7) Next Steps Confirmation

After presenting findings, **MUST** ask user how to proceed. Do **NOT** implement changes until user explicitly confirms:

```
---

## Next Steps

Found X issues (P0: _, P1: _, P2: _, P3: _).

**How would you like to proceed?**

1. **Fix all** — Implement all suggested fixes
2. **Fix P0/P1 only** — Address critical and high priority issues only
3. **Fix specific items** — Tell me which issues to fix
4. **No changes** — Review complete, no implementation needed

Please choose an option or provide specific instructions.
```

## Behavior Rules

1. **Review-first**: Default to review output only; do not modify code until user confirms.
2. **Be specific**: Always reference `file:line` for each finding.
3. **Python examples**: All fix suggestions must use Python syntax.
4. **Strengths first**: List what's working well before diving into issues.
5. **Prioritize**: P0 → P1 → P2 → P3, critical security issues first.
6. **Scope-appropriate**: Don't over-engineer simple scripts; don't under-engineer production code.
7. **Tool recommendations**: End every review with relevant static analysis tool suggestions.

## References

| File | Purpose |
|------|---------|
| `references/solid-checklist.md` | SOLID smell prompts and refactor heuristics |
| `references/security-checklist.md` | General web/app security and runtime risk checklist |
| `references/python-security-checklist.md` | Python-specific: deserialization, SSTI, command injection, path traversal |
| `references/code-quality-checklist.md` | Error handling, performance, caching, boundary conditions |
| `references/python-anti-patterns.md` | Python-specific: mutable defaults, late binding, asyncio, GIL, PEP8 |
| `references/removal-plan.md` | Dead code deletion template and follow-up plan |

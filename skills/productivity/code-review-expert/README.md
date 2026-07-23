# Code Review Expert

A comprehensive Python code review skill for AI agents. Performs structured reviews with a senior engineer lens, covering architecture, security (general + Python-specific), performance, and code quality.

## Origin

Integrated from multiple upstream skills:

| Source | What's Adopted |
|--------|---------------|
| [sanyuan0704/code-review-expert](https://github.com/sanyuan0704/sanyuan-skills) | 7-step workflow, P0-P3 severity, SOLID + architecture review, detailed security checklist, dead code removal planning, performance & boundary conditions |
| [yennanliu/python-code-reviewer](https://github.com/yennanliu/ai_experiment) | Strengths section, 1-10 rating, Python idioms awareness, Python code example requirements |
| Custom additions | Python-specific security (pickle/eval/subprocess/SSTI/path traversal), Python anti-patterns (mutable defaults, late binding, asyncio,GIL), toolchain integration (ruff/bandit/mypy) |

## Features

- **SOLID Principles** — Detect SRP, OCP, LSP, ISP, DIP violations + code smells + refactor heuristics
- **Security Scan** — General: injection, SSRF, race conditions, auth gaps, JWT, crypto, secrets leakage
- **Python Security** — Pickle deserialization, eval/exec injection, subprocess injection, SSTI, path traversal quirks
- **Performance** — N+1 queries, missing indexes, CPU hotspots, missing cache, unbounded memory, no pagination
- **Error Handling** — Swallowed exceptions, async error gaps, overly broad catch, resource management
- **Boundary Conditions** — None handling, empty collections, numeric boundaries, off-by-one
- **Python Anti-Patterns** — Mutable defaults, late binding closures, asyncio pitfalls, GIL awareness, class attribute confusion
- **PEP 8 Compliance** — Naming, whitespace, import order, line length, type annotations
- **Removal Planning** — Identify dead code with safe deletion plans and deferral strategies
- **Review-First** — Output-only by default; user must explicitly approve before any changes are made

## Installation

The skill is installed as part of the `pi-common-skills` repository:

```
git clone git@github.com:4cya/pi-common-skills.git
```

The skill is located at `productivity/code-review-expert/` within the repo and will be auto-discovered by Pi.

## Usage

After installation, trigger the skill in Pi by saying:

- `review my code`
- `code review`
- `/code-review-expert`

Or any similar code review request. The skill automatically scopes git changes via `git diff`.

## Workflow

1. **Preflight** — Scope changes via `git diff`. Handle edge cases (empty diff, large diffs >500 lines, mixed concerns).
2. **SOLID + Architecture** — Check design principles and code smells against `references/solid-checklist.md`.
3. **Removal Candidates** — Find dead/unused code with `references/removal-plan.md` template.
4. **Security Scan** — General vulnerabilities (`references/security-checklist.md`) + Python-specific (`references/python-security-checklist.md`).
5. **Code Quality** — Error handling, performance, boundaries (`references/code-quality-checklist.md`) + Python anti-patterns (`references/python-anti-patterns.md`).
6. **Output** — Findings by severity (P0-P3) + Strengths + 1-10 rating + tool recommendations.
7. **Confirmation** — Present next-step options; do not modify code until user confirms.

## Severity Levels

| Level | Name | Action |
|-------|------|--------|
| P0 | Critical | Must block merge |
| P1 | High | Should fix before merge |
| P2 | Medium | Fix or create follow-up |
| P3 | Low | Optional improvement |

## Structure

```
code-review-expert/
├── SKILL.md                          # Main skill definition (workflow + output format)
├── README.md                         # This file
└── references/
    ├── solid-checklist.md            # SOLID smell prompts + refactor heuristics
    ├── security-checklist.md         # General web/app security + runtime risks
    ├── python-security-checklist.md  # Python-specific: deserialization, SSTI, injection, paths
    ├── code-quality-checklist.md     # Error handling, performance, boundary conditions
    ├── python-anti-patterns.md       # Python-specific: mutable defaults, asyncio, GIL, PEP8
    └── removal-plan.md              # Dead code deletion template
```

## License

MIT

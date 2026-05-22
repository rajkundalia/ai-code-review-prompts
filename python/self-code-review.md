# Python Self Code Review

Review my uncommitted changes (or the files I point you to) as if you are a **chief programmer** and a **chief architect** reviewing my code before I open a PR. Be blunt. I want to catch problems before a reviewer does.

**Reference files** — load these before starting and use them as the standard:
- `best-practices.md` — coding conventions, style, testing, error handling
- `library-guidelines.md` — library-specific patterns and gotchas (requests, etc.)

---

## Step 1 — Scope

Look at my changes (`git diff` or files I specify). Summarize:

- What did I change and why (infer from the diff)
- Production code vs test code vs config/infra

Present findings. **Stop and wait** for me to confirm before moving on.

---

## Step 2 — Correctness & Completeness

- Does the code actually do what it claims? Trace the logic.
- Are there edge cases I didn't handle?
- Are there error paths that silently pass or swallow exceptions?
- If I changed production code, did I update or add tests?
- If I changed tests, do they actually test the real flow — or am I mocking away the thing I should be validating?
- Am I testing stdlib behavior? Dataclass constructors, frozen immutability, StrEnum values with no custom logic don't need dedicated tests — they'll be exercised by higher-level tests.

Present findings. **Stop and wait** for me to confirm before moving on.

---

## Step 3 — Standards Check

Check against **`best-practices.md`** and **`library-guidelines.md`**:

- Type hints: modern syntax, no legacy `typing` imports
- Functions: ≤20 lines, return early, keyword-only args for 3+ params
- Naming: no single-letter variable names — use descriptive names that convey intent
- Error handling: specific exceptions, `from e` chaining, no bare `except`
- Logging: lazy formatting, no f-strings in log calls, no `print()`
- `pathlib.Path` over string manipulation
- pytest idioms: fixtures, parametrize, mock at boundaries only
- Library usage: check `library-guidelines.md` for any libraries touched
- No dead code, no TODOs without linked issues, no commented-out blocks

Present findings. **Stop and wait** for me to confirm before moving on.

---

## Step 4 — Devil's Advocate

Pretend you are the harshest reviewer on the team. Ask:

- What is the laziest thing in this diff?
- What will break first in production?
- Is any change here a convenience hack that doesn't belong (e.g., test scaffolding leaking into prod)?
- If someone reads this code 6 months from now with no context, what will confuse them?

Present findings. **Stop and wait** for me to confirm before moving on.

---

## Step 5 — Verdict

Give me a punch list:

- **Fix now** — things that will get rejected in review or cause bugs
- **Fix before merge** — real issues, but won't block a first review round
- **Consider** — suggestions, not requirements

Be specific: file, line, what's wrong, what to do instead.

# Python 3.13 Project Conventions

## Principles

- Write code that is obvious to read, not clever to write.
- Every function does one thing. If you need the word "and" to describe it, split it.
- Fail fast, fail loud. No silent swallowing of errors.
- Design for testability: inject dependencies, keep I/O at the edges.

---

## Style & Structure

- PEP 8 enforced via `ruff`. Max line length: 120. Target: `py313`.
- Trailing commas in all multi-line structures.
- Absolute imports only. No wildcard imports. No unused imports.
- Remove dead code — no commented-out blocks, no `TODO` without a linked issue.
- Project layout: source in `src/<package>/`, tests in `tests/` mirroring `src`, shared fixtures in `conftest.py`.

---

## Type Hints (Mandatory on all public interfaces)

- Modern 3.13 syntax only: `list[str]`, `X | None`, type aliases, inline generics `def f[T](x: T) -> T:`.
- No legacy imports from `typing` for builtins (`List`, `Dict`, `Optional`, `Tuple`).
- Use `TypeIs` over `TypeGuard`. Use `TypeVar` defaults where appropriate.
- `# type: ignore` must include error code: `# type: ignore[attr-defined]`.

---

## Functions & Design

- ≤20 lines per function. Return early to reduce nesting. Exceptions require a comment explaining why.
- 3+ parameters → use keyword-only args (`*`). For 2-param functions, use `*` when argument order is non-obvious or types are identical. Exceptions require a comment explaining why.
- Never use mutable defaults (`def f(items=[]):`). Use `None` and assign inside.
- Google-style docstrings on public functions. Don't restate type hints in docstrings.
- Pure functions where possible. Keep I/O and side effects at the boundaries.
- Prefer composition over inheritance. `@dataclass` or Pydantic models for data containers.
- Anything that is common and repetitive across functions should be extracted.

---

## Error Handling & Logging

- Catch specific exceptions only. No bare `except:`. No `except Exception: pass`.
- Chain with `from e` when wrapping. Handle errors at one layer, not multiple.
- `logging` module only, never `print()`. Lazy formatting: `logger.info("user %s", uid)` — no f-strings in log calls.
- Never log secrets, tokens, or credentials. Sanitize before logging.

---

## Async

- `asyncio.run()` as entry point. Never manage loops manually.
- Prefer `TaskGroup` over `gather()`. Be aware: if one task raises, `TaskGroup` cancels all remaining tasks before re-raising — unlike `gather(return_exceptions=True)`, which lets all tasks run to completion. Choose accordingly.
- `asyncio.timeout()` on all network calls.
- `asyncio.to_thread()` for blocking I/O. Never call blocking code inside `async def`.
- Re-raise `CancelledError` after cleanup. Use `except*` for `ExceptionGroup`.

---

## Security & Config

- All secrets via environment variables or `.env`. Never hardcode. Never commit `.env`.
- Validate config at startup with `os.environ["KEY"]` (fails loud) — not deep in call stacks.
- Don't use `assert` for runtime validation — stripped with `-O`. Use `if`/`raise ValueError`.
- `.gitignore`: `.env`, `__pycache__/`, `*.pyc`, `.mypy_cache/`, `.pytest_cache/`, `htmlcov/`, `dist/`.

---

## Testing

- **Arrange → Act → Assert**, separated by blank lines.
- Name: `test_<does_what>_when_<condition>`. Group related tests in classes.
- Mock at boundaries (external services, I/O), never internal implementation.
- `AsyncMock` for async functions. Inject dependencies rather than patching globally.
- `pytest.raises(SpecificError, match="...")` for exception tests.
- `@pytest.mark.parametrize` with `ids=` for multiple cases.
- Test error paths, edge cases, and empty inputs — not just the happy path.
- Don't test stdlib behavior: no tests for plain dataclass constructors, frozen immutability, `StrEnum` values, or type assignments when the model has no custom logic. These models get tested through actual usage in higher-level tests.
- Never use `==` for float comparison — use `pytest.approx()`. Floating point arithmetic is imprecise.
- Shared fixtures in `conftest.py`. Factory fixtures for test data with variations.
- ≥80% meaningful coverage. Don't write tests just to hit a number.
- `asyncio_mode = "auto"` is set globally — do not add `@pytest.mark.asyncio` to individual tests.

---

## Tooling

```toml
[tool.ruff]
line-length = 120
target-version = "py313"

[tool.ruff.lint]
select = ["E", "F", "W", "I", "N", "UP", "ANN", "B", "SIM", "RUF"]
# ANN enforces type annotations on public interfaces.
# UP catches legacy typing imports (List, Dict, Optional, etc.).

[tool.pytest.ini_options]
testpaths = ["tests"]
addopts = "-v --strict-markers --tb=short --cov=src/ --cov-report=term-missing"
asyncio_mode = "auto"

[tool.mypy]
python_version = "3.13"
strict = true
warn_return_any = true
```

---

## Reference

Library-specific patterns and gotchas: see `library-guidelines.md`

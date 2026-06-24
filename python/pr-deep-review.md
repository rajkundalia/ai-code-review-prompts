# Python PR Deep Review

Review PR #{number} using the phased workflow below. Do not skip phases. Do not start the next phase until I say so.

**Reference files** — load these before starting and use them as the standard for Phase 3:
- `best-practices.md` — coding conventions, style, testing, error handling
- `library-guidelines.md` — library-specific patterns and gotchas (requests, etc.)

---

## Phase 1 — Understand

Get the PR diff and details. Then give me:

- One-paragraph summary of what this PR does
- List of production code changes vs test-only changes
- Any external/infrastructure changes (CI, config, dependencies)

Present findings. **Stop and wait.** I will ask questions to build my understanding before we go further.

---

## Phase 2 — Surface Review

Review the PR for obvious issues. Flag:

- TODOs, FIXMEs, or temp code left in
- Security concerns (hardcoded secrets, injection risks, exposed filesystem)
- Dependency changes and their blast radius
- Config/CI changes that affect more than this PR intends
- Naming, spelling, or copy-paste errors

Present findings. **Stop and wait** for me to confirm or dismiss before moving on.

---

## Phase 3 — Deep Review

Think like a **chief programmer** and a **chief architect**. Review the diff independently - Q&A from earlier phases was for *my* understanding, not a substitute this review. For each area below, state your finding and your reasoning.

### 3a. Production Code

- Does each change serve the stated PR goal, or is it a convenience hack for something else (e.g., test scaffolding leaking into prod)?
- Are there behavioral side effects for existing callers?
- Is the change safe to roll back independently?

### 3b. Tests — Real Coverage vs Theater

This is the most important section. For every test, answer:

1. **Does this test exercise the real flow?** Trace the call from the test through to the code under test. If the implementation changes in a meaningful way, will this test fail? If not, it's not testing anything.
2. **Is it mocking away the thing it should be testing?** Mocks are for isolating boundaries (network, disk, external services). If a test mocks the function it's supposed to validate, flag it. Spies that wrap the real function and capture args are fine.
3. **What breaks this test?** List the specific production changes that would cause a failure. If the answer is "almost nothing" or "only a rename", the test has low value.
4. **Is it testing stdlib?** Flag tests that only verify dataclass constructors, frozen immutability, StrEnum values, or type assignments on models with no custom logic. These are testing Python, not the project.

### 3c. Python Best Practices

Check against **`best-practices.md`** and **`library-guidelines.md`**. Key areas:

- Type hints: modern 3.13 syntax (`list[str]`, `X | None`), no legacy `typing` imports
- Functions: ≤20 lines, return early, keyword-only args for 3+ params
- Naming: no single-letter variable names — use descriptive names that convey intent (e.g., `result` not `r`, `directory` not `d`)
- Error handling: specific exceptions only, `from e` chaining, no bare `except`
- Logging: lazy formatting (`logger.info("msg %s", arg)`), never f-strings, never `print()`
- `pathlib.Path` over string manipulation for filesystem ops
- Proper use of pytest idioms: fixtures, parametrize, `tmp_path`, `monkeypatch`
- Mock at boundaries only, never internal implementation
- Library usage: check `library-guidelines.md` for any libraries used in the PR (e.g., requests timeout enforcement, session patterns)

### 3d. Devil's Advocate

Challenge the PR from the perspective of someone who thinks it should NOT be merged. Ask:

- What is the strongest argument against merging this as-is?
- What is the most likely way this PR causes a problem 3 months from now?
- If you had to cut 40% of this PR to ship faster, what would you cut and why?

Present findings. **Stop and wait.**

---

## Phase 4 — Verdict

After all phases, give a final summary:

- **Must fix before merge** — blocking issues
- **Should fix** — real problems that can be followed up but are not blocking
- **Nitpicks** — style or preference, take-or-leave

End with a clear **APPROVE**, **REQUEST CHANGES**, or **NEEDS DISCUSSION** recommendation.

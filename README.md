# ai-code-review-prompts

Prompts I use to review Python code — on my own diff before raising a PR, and on PRs during review.

Tool-agnostic; paste into any LLM. Iterated on with Claude Code over ~2 months of real PR reviews.

## Python

| File | Type | Purpose |
|---|---|---|
| [`python/pr-deep-review.md`](python/pr-deep-review.md) | Prompt | Phased deep review of a PR. |
| [`python/self-code-review.md`](python/self-code-review.md) | Prompt | Deep review of your own code before raising a PR. |
| [`python/best-practices.md`](python/best-practices.md) | Reference | Python conventions loaded by the prompts above. |
| [`python/library-guidelines.md`](python/library-guidelines.md) | Reference | Library-specific patterns loaded by the prompts above. |

## How to use

1. Copy the prompt you need into your AI tool (Claude, ChatGPT, Cursor, Claude Code, etc.).
2. When the prompt's **Reference files** block lists `best-practices.md` and `library-guidelines.md`, load those alongside so the prompt can use them as the standard.
3. Follow the phases. The prompts stop and wait between phases — don't skip ahead.

## How I use these

Full workflow, including the "one session per PR" technique and how to keep your own understanding in the loop → [Medium article]({MEDIUM_URL})

## Contributing

Suggestions and PRs welcome. To add prompts for a new language, create a new folder (e.g., `javascript/`, `go/`). To improve existing prompts, open a PR — keep it focused and explain the change in real-world terms ("I hit X problem; this tightens Y").

## License

Free to use, modify, and share.

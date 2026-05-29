# claude-md-guidelines

A crisp, modular, **general-purpose `CLAUDE.md`** you can drop into any project to make AI coding agents — Claude or any other — make fewer mistakes. It does this by being explicit about how to *think*, how to *build*, and how to *verify*, so the agent stops guessing and starts working to a standard.

Vendor-neutral and stack-agnostic by design. Nothing here is tied to a company, framework, or language.

## What's inside

| File | What it is |
|---|---|
| [`CLAUDE.md`](CLAUDE.md) | The crisp entry point: the core operating loop + links to the guides. This is the file your agent reads. |
| [`guides/working-principles.md`](guides/working-principles.md) | How to approach any task — think before coding, simplicity first, surgical changes, goal-driven execution. |
| [`guides/engineering-standards.md`](guides/engineering-standards.md) | The quality bar while writing code — context-first, production engineering, naming, comments, dependency & security discipline. |
| [`guides/review-and-verification.md`](guides/review-and-verification.md) | How to check the work before shipping — severity-ordered review, final audit, adding guards safely, verification & testing. |

## How to use

1. Copy `CLAUDE.md` and the `guides/` folder into the root of your project.
2. Your coding agent reads `CLAUDE.md` automatically and follows the links as needed.
3. Add your project-specific rules in a clearly marked section of `CLAUDE.md`, or in a separate file it references — keep the shared guidelines and your local rules distinct.
4. Tune to taste. These are strong defaults, not laws; override any rule explicitly when your project has a good reason to.

## Philosophy

- **Caution over speed.** Surface assumptions and tradeoffs before writing code.
- **Surgical over sweeping.** Every changed line traces to the request.
- **Verify, don't assume.** Define success criteria and check them.
- **Defaults, not dogma.** Adapt the rules; the goal is fewer mistakes, not ceremony.

## Contributing

PRs welcome. Keep every contribution **general and vendor-neutral** — no company, product, framework, or stack-specific references. If a rule only makes sense for one ecosystem, it belongs in a project's own `CLAUDE.md`, not here.

## License

[MIT](LICENSE).

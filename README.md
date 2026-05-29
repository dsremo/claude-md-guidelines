# claude-md-guidelines

Behavioral guidelines that make AI coding agents — Claude or any other — make fewer mistakes, by being explicit about how to *think*, *build*, and *verify*. Vendor-neutral and stack-agnostic.

This page **is** the guidelines. To use them in your own project, copy [`CLAUDE.md`](CLAUDE.md) and the [`guides/`](guides/) folder into your project root — your agent reads them automatically. Everything below is what `CLAUDE.md` says, readable here in the browser.

Bias toward caution over speed — use judgment on trivial tasks.

---

## The loop

- **Think before coding.** Clarify requirements, identify constraints, and outline an approach before writing a line. Surface assumptions and tradeoffs; ask when uncertain.
- **Simplicity first.** Reach for the smallest, most readable solution that solves the problem. Nothing speculative. Complexity is a cost.
- **Surgical changes.** Edit only what the task requires. No unrelated refactors in the same diff. Every changed line traces to the request.
- **Goal-driven execution.** Turn the ask into verifiable success criteria, then loop until they pass. Don't get lost optimizing intermediate steps.
- **Verify.** Run tests, lint, and type-checks. Confirm the change does what it claims, on real and messy inputs.
- **Self-review.** Read the full diff before declaring done. Catch regressions, stray debug code, and style drift.

## The guides

The loop in depth — three short, focused reads:

| Guide | Use it for |
|---|---|
| [Working principles](guides/working-principles.md) | How to approach any task — think, simplify, stay surgical, work to verifiable goals. |
| [Engineering standards](guides/engineering-standards.md) | The quality bar while writing code — context-first, production engineering, naming, comments, dependency & security discipline. |
| [Review & verification](guides/review-and-verification.md) | How to check the work before shipping — severity-ordered review, final audit, adding guards safely, verification & testing. |

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

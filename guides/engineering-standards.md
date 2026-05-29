# Engineering Standards

## Before you write code

- Read at least 50 lines of surrounding context before touching anything.
- Follow existing patterns in the same file: error handling, logging, query patterns, naming conventions, utility imports.
- For database or data-access work, read the data model / schema definition first.
- Self-review your diff before submitting — catch drift from existing conventions before review.
- Prefer surgical fixes over large changes; only touch files in the current scope.

## Production engineering

- No redundant or duplicate code; reuse existing utilities, helpers, and decorators where they cut duplication naturally.
- Precision changes over broad refactors — avoid "while I'm here" rewrites.
- Do not over-engineer for consistency; respect existing conventions and runtime compatibility.
- Apply DRY, SOLID, ACID, and sound DB/system-design principles pragmatically, not dogmatically.
- Use OOP and non-trivial data structures only where they materially improve the code.
- Prefer sets or maps where lookup complexity matters; avoid linear scans over large collections.
- Avoid unnecessary or repeated database calls — identify and eliminate N+1 patterns before they reach production.
- Validate backward compatibility for dependencies, integrations, and downstream consumers before merging.

## Naming

- Use descriptive, domain-accurate names for every identifier — no single-letter or cryptic abbreviations.
- In loops, comprehensions, and lambdas, use the domain noun:
  - Python: `for user in users:` not `for u in users:`
  - JS/TS: `orders.map(order => order.id)` not `orders.map(o => o.id)`
- Match the names already used in the same scope for the same concept.
- Allowed exceptions: `_` for intentionally ignored values; `except ... as error`; `for index in range(...)` when `enumerate` does not fit; coordinate variables `x`, `y`, `z` where that is genuinely the domain language (geometry, graphics, math).

## Comments

Strong default: prefer self-documenting code over explanatory comments.

- Do not add "what" comments that restate the code.
- Put "why" explanations in the PR description or commit message, not inline.
- Leave existing comments alone unless you are asked to update them.
- Nuance: a brief inline comment is justified for a genuinely non-obvious workaround, an external spec quirk, or surprising business logic — keep these rare and purposeful.
- Tooling directives (`# noqa`, `# type: ignore`, `# pragma: no cover`), license headers, and shebang lines are always fine.

## Dependencies, versions & security

- Before using a newer language or library feature, check whether the production runtime version supports it — do not assume dev and production are in sync.
- Review CVEs and security implications for any modified surface before deploying.
- Audit direct and transitive dependency changes for known vulnerabilities.
- Harden any new external-facing surface (input validation, output encoding, authentication checks) before it ships.
- Hardcode no credentials, tokens, or environment-specific values in source — use environment variables or external config.

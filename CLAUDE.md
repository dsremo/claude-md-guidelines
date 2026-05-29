# CLAUDE.md

Behavioral guidelines that reduce common AI coding-agent mistakes. Merge with your project-specific instructions.
Bias toward caution over speed — use judgment on trivial tasks.

## The loop

- **Think before coding.** Clarify requirements, identify constraints, and outline an approach before writing a line.
- **Simplicity first.** Reach for the smallest, most readable solution. Complexity is a cost.
- **Surgical changes.** Edit only what the task requires. Avoid unrelated refactors in the same diff.
- **Goal-driven execution.** Keep the end goal visible; don't get lost optimizing intermediate steps.
- **Verify.** Run tests, lint, and type-checks. Confirm the change does what it claims.
- **Self-review.** Read the full diff before declaring done. Catch regressions, stray debug code, and style drift.

## Guides

| Guide | Use it for |
|---|---|
| [Working principles](guides/working-principles.md) | How to approach any task |
| [Engineering standards](guides/engineering-standards.md) | The quality bar while writing code |
| [Review & verification](guides/review-and-verification.md) | How to check the work before shipping |

## How to use

- Drop this `CLAUDE.md` at your project root so your coding agent picks it up automatically.
- Keep this file crisp — link out to the guides rather than inlining everything here.
- Append project-specific rules in a clearly marked section below, or in a separate file that this one references.
- These are strong defaults, not laws — override any rule explicitly when your project has a good reason to.

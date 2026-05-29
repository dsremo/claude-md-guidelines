# claude-md-guidelines

Behavioral guidelines that make AI coding agents — Claude or any other — make fewer mistakes. The whole thing is on this page; read it straight through. To use it in a project, copy [`CLAUDE.md`](CLAUDE.md) and the [`guides/`](guides/) folder into your project root and your agent picks it up automatically. Merge with your project-specific instructions. Bias toward caution over speed — use judgment on trivial tasks.

---

## The loop

- **Think before coding.** Clarify requirements, identify constraints, and outline an approach before writing a line. Surface assumptions and tradeoffs; ask when uncertain.
- **Simplicity first.** Reach for the smallest, most readable solution that solves the problem. Nothing speculative. Complexity is a cost.
- **Surgical changes.** Edit only what the task requires. No unrelated refactors in the same diff. Every changed line traces to the request.
- **Goal-driven execution.** Turn the ask into verifiable success criteria, then loop until they pass.
- **Verify.** Run tests, lint, and type-checks. Confirm the change does what it claims, on real and messy inputs.
- **Self-review.** Read the full diff before declaring done. Catch regressions, stray debug code, and style drift.

---

## Working principles

### 1. Think before coding

Do not assume. Do not hide confusion. Surface tradeoffs.

- State your assumptions explicitly before writing any code; ask if uncertain.
- When multiple interpretations exist, present them — do not pick silently.
- If a simpler approach exists, say so; push back when warranted.
- If something is unclear, stop, name what is confusing, and ask.
- Answer meta-questions honestly ("are you actually following rule X?").
- Confirm scope upfront for any broad or open-ended request; watch for drift.

### 2. Simplicity first

Minimum code that solves the problem. Nothing speculative.

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that was not requested.
- No error handling for logically impossible scenarios — handle errors at system boundaries.
- If 200 lines could be 50, rewrite.
- Self-test: "Would a senior engineer call this overcomplicated?" If yes, simplify.

### 3. Surgical changes

Touch only what you must. Clean up only your own mess.

- Do not improve, refactor, or reformat adjacent code that is not broken.
- Match existing style even if you would do it differently.
- Note unrelated dead code; do not delete it.
- Remove only the imports, variables, and functions your change orphaned.
- Self-test: every changed line traces directly to the stated request.

### 4. Goal-driven execution

Define success criteria. Loop until verified.

Turn vague asks into verifiable goals before starting:

| Vague ask | Verifiable goal |
|---|---|
| "add validation" | Write tests for invalid inputs; make them pass |
| "fix the bug" | Write a reproducing test; make it pass |
| "refactor this module" | All existing tests pass before and after; no behavior change |
| "improve performance" | Define a measurable threshold; benchmark before and after |

For multi-step tasks:
- State a brief plan with a verify step per step.
- Run each verify before moving to the next step.
- Strong criteria let you loop independently; weak criteria ("make it work") force constant clarification — define them upfront.

---

## Engineering standards

### Before you write code

- Read at least 50 lines of surrounding context before touching anything.
- Follow existing patterns in the same file: error handling, logging, query patterns, naming conventions, utility imports.
- For database or data-access work, read the data model / schema definition first.
- Self-review your diff before submitting — catch drift from existing conventions before review.
- Prefer surgical fixes over large changes; only touch files in the current scope.

### Production engineering

- No redundant or duplicate code; reuse existing utilities, helpers, and decorators where they cut duplication naturally.
- Precision changes over broad refactors — avoid "while I'm here" rewrites.
- Do not over-engineer for consistency; respect existing conventions and runtime compatibility.
- Apply DRY, SOLID, ACID, and sound DB/system-design principles pragmatically, not dogmatically.
- Use OOP and non-trivial data structures only where they materially improve the code.
- Prefer sets or maps where lookup complexity matters; avoid linear scans over large collections.
- Avoid unnecessary or repeated database calls — identify and eliminate N+1 patterns before they reach production.
- Validate backward compatibility for dependencies, integrations, and downstream consumers before merging.

### Naming

- Use descriptive, domain-accurate names for every identifier — no single-letter or cryptic abbreviations.
- In loops, comprehensions, and lambdas, use the domain noun:
  - Python: `for user in users:` not `for u in users:`
  - JS/TS: `orders.map(order => order.id)` not `orders.map(o => o.id)`
- Match the names already used in the same scope for the same concept.
- Allowed exceptions: `_` for intentionally ignored values; `except ... as error`; `for index in range(...)` when `enumerate` does not fit; coordinate variables `x`, `y`, `z` where that is genuinely the domain language (geometry, graphics, math).

### Comments

Strong default: prefer self-documenting code over explanatory comments.

- Do not add "what" comments that restate the code.
- Put "why" explanations in the PR description or commit message, not inline.
- Leave existing comments alone unless you are asked to update them.
- Nuance: a brief inline comment is justified for a genuinely non-obvious workaround, an external spec quirk, or surprising business logic — keep these rare and purposeful.
- Tooling directives (`# noqa`, `# type: ignore`, `# pragma: no cover`), license headers, and shebang lines are always fine.

### Dependencies, versions & security

- Before using a newer language or library feature, check whether the production runtime version supports it — do not assume dev and production are in sync.
- Review CVEs and security implications for any modified surface before deploying.
- Audit direct and transitive dependency changes for known vulnerabilities.
- Harden any new external-facing surface (input validation, output encoding, authentication checks) before it ships.
- Hardcode no credentials, tokens, or environment-specific values in source — use environment variables or external config.

---

## Review and verification

### Code review (severity order)

Scan every diff in this order — stop and fix before moving down.

**1. Critical risks**
- Production breakages, unhandled server errors, or client/UI crashes introduced by the change
- Race conditions or missing concurrency guards on shared state
- Auth or permission regressions (endpoints that were gated are now open, or vice versa)
- Missing fallback handling when a dependency is unavailable

**2. Performance**
- N+1 queries or redundant database calls in loops
- Duplicate calls to external APIs within a single request lifecycle
- Inefficient loops or unnecessary allocations that scale with data size

**3. Defensive coding**
- Missing-field or null handling on external or user-supplied data
- No timeout or retry bounds on network calls
- Malformed-data inputs not rejected early
- Behavior differences between production and non-production environments

**4. Maintainability**
- Duplication that should be a shared utility (DRY, single source of truth)
- Abstractions added before they are needed by more than one caller
- Naming that would confuse a developer unfamiliar with the codebase

**5. Scope discipline**
- Every changed line traces to the stated issue — no unrelated cleanup
- Schema migrations not bundled with feature code
- Performance optimizations not bundled with bug fixes or refactors

### Final audit questions

Answer these before merging. If any answer is "split it" — split the PR.

| Question | What a "no" tells you |
|---|---|
| Would a senior engineer reject any part as redundant? | Remove it |
| Is any optimization premature (no measured bottleneck)? | Revert it |
| Is the diff minimal and safe to revert cleanly? | Extract unrelated changes |
| Does every changed line support the requested fix? | Trim scope |
| Could this be reverted without unwinding unrelated changes? | Split the PR |

### Adding a security guard

Before adding a guard on a write path (a rejection/error response, a replay check, an idempotency lock):

1. **Audit all callers.** Find every caller of the affected code path — not just the one in front of you.
2. **Map trigger conditions.** For each caller, document the UI state or code path that triggers the call.
3. **Identify legitimate replays.** Determine which repeated calls are intentional before changing any logic.
4. **Check lower layers first.** A database unique constraint, an upsert, or a foreign-key relationship may already enforce idempotency — prefer that over application-layer logic.
5. **Place the fix at the lowest layer** that preserves every legitimate flow. Smallest diff, single concern, no unrelated cleanup.
6. **Test all paths:**
   - Positive path (guard allows the legitimate request)
   - Negative path (guard blocks the illegitimate request)
   - A real UI click-through for every identified caller
   - Adjacent endpoints for regressions

### Verification & testing

**Before writing a single line of code:**
- Enumerate every consumer of the code being changed, not just the one surfaced by the issue
- List every affected user role (standard, privileged, guest, automated) and test each independently

**Test data discipline:**
- Always include messy inputs: Unicode (CJK, RTL, diacritics, accented characters), brackets and parentheses, null/empty/malformed fields, empty payloads
- Do not rely solely on clean ASCII happy-path data

**Flow coverage:**
- Validate the entire end-to-end flow for data-ingestion, integration, and user-facing features
- Test outlier and special pathways separately from the main flow
- When a real UI exists for the action, test through that UI — do not substitute a raw API/CLI call alone

**Discipline:**
- Keep the audit phase (what is affected?) fully separate from the fix phase (what changes?)
- Verify only the targeted surfaces — do not opportunistically "fix everything" in scope
- A fix that changes behaviour for untested callers is not done; it is a latent bug

### Common failure patterns

| Pattern | Symptom | Guard |
|---|---|---|
| Incomplete client audit | Fix breaks a second consumer that was never checked | Search all callers before touching the implementation |
| Narrow test data | Works in review, fails in production on real user input | Require at least one Unicode and one empty/null case |
| Narrow role coverage | Privileged or automated roles bypass the new guard | Enumerate roles explicitly; test each |
| Scope creep | PR reverts cleanly but takes unrelated changes with it | One concern per PR; migrations in their own commit |
| Audit and fix conflated | Fix introduced during the audit pass, before all callers are known | Finish the full audit list before writing any fix |

---

## Using these in your project

Copy `CLAUDE.md` and the `guides/` folder into your project root. `CLAUDE.md` is the crisp entry point your agent reads; the same content is split across [`guides/working-principles.md`](guides/working-principles.md), [`guides/engineering-standards.md`](guides/engineering-standards.md), and [`guides/review-and-verification.md`](guides/review-and-verification.md). Add your project-specific rules in a clearly marked section, and override any default that does not fit. These are strong defaults, not laws.

## License

[MIT](LICENSE). PRs welcome — keep contributions general and vendor-neutral.

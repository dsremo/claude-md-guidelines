# Review and verification

## Code review (severity order)

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

---

## Final audit questions

Answer these before merging. If any answer is "split it" — split the PR.

| Question | What a "no" tells you |
|---|---|
| Would a senior engineer reject any part as redundant? | Remove it |
| Is any optimization premature (no measured bottleneck)? | Revert it |
| Is the diff minimal and safe to revert cleanly? | Extract unrelated changes |
| Does every changed line support the requested fix? | Trim scope |
| Could this be reverted without unwinding unrelated changes? | Split the PR |

---

## Adding a security guard

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

---

## Verification & testing

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

---

## Common failure patterns

| Pattern | Symptom | Guard |
|---|---|---|
| Incomplete client audit | Fix breaks a second consumer that was never checked | Search all callers before touching the implementation |
| Narrow test data | Works in review, fails in production on real user input | Require at least one Unicode and one empty/null case |
| Narrow role coverage | Privileged or automated roles bypass the new guard | Enumerate roles explicitly; test each |
| Scope creep | PR reverts cleanly but takes unrelated changes with it | One concern per PR; migrations in their own commit |
| Audit and fix conflated | Fix introduced during the audit pass, before all callers are known | Finish the full audit list before writing any fix |

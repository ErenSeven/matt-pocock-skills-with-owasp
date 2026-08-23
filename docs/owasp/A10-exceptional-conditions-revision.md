# OWASP A10:2025 — Mishandling of Exceptional Conditions — Skill Revision Document (project-independent)

Purpose: Itemize A10 under three headings (causes — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A10 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A10:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: A10 is a **new** category in OWASP Top 10:2025 (replacing 2021's SSRF). Highly code-visible — error handling, fail-closed, resource cleanup, transaction rollback, race conditions. The exception-handling/fail-closed/rollback items deliberately left here from A09 now find their true home. **Final category — the Top 10 pass is completed with this one.**

Cross-references (PDF): A10 ↔ A06 (design/threat modeling), A10 ↔ A09 (log/alert). Additionally: A10 ↔ A02 (centralized error handling / stack-trace leakage), A10 ↔ A05 (input validation), A10 ↔ A01 (rate limiting).

## 0. Cross-ref and out-of-scope items

Owned by another category (A10 does not re-own them, cross-ref only):

| # | Item | Owner | Note |
|---|---|---|---|
| S2 | DB error shown in full to the user → SQLi reconnaissance | **A02** (stack-trace/error leakage) | A10 says "handle the error"; "don't leak the error to the user" is in A02 + `implement`'s Secure defaults |
| O3 / O6 | Monitor recurring error patterns; summarize high error rates | **A09** (logging/monitoring emission) | A10 logs the error; monitoring/summarization is A09 + ops |
| O8 (threat-modeling) | Threat modeling | **A06** (Insecure Design) | Design-time; the A06 Threat Model section |

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| O8 (stress/pentest part) | Stress testing / pentest | Separate test-operation/tooling |
| O9 | Consistent approach across the organization | Org process |

## 1. Causes & preventions (in scope: 5/9, partial: 2, cross/out: 2)

| # | A10 prevention | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| O1 | Catch every error at its origin; handle it meaningfully | ❌ | ✅ | `audit-security` (A10), `code-review`, `implement`, `tdd` | Catch-at-origin; swallowed/empty-catch anti-pattern; meaningful handling |
| O2 | Global exception handler | ❌ | ✅ | `implement`, `code-review` | Route to a central handler (A02 cross-ref — without leaking stack traces) |
| O4 | Error mid-transaction → roll back the whole thing (fail closed) | ❌ | ✅ | `audit-security` (A10), `code-review`, `implement`, `tdd` | Atomicity/rollback; fail-closed on error (inherited from A09) |
| O5 | Rate limiting/resource quota/throttling (no limit = DoS/cloud-bill) | ❌ | ✅ | `audit-security` (A10), `code-review`, `implement` | Throttle/quota for expensive operations (the resilience angle of A01's rate-limit) |
| O7 | Strict input validation + centralized error handling | ❌ | ✅ | `implement` | Centralized, consistent error handling (A05 validation + A02 handler cross-ref) |
| O3 (partial) | Monitor recurring errors | ❌ | 🟡 | `implement` | Emit the error; A09 logs it |
| O6 (partial) | Summarize high error rates | ❌ | 🟡 | `implement` | Compatible emission with A09 |
| O8 | threat modeling / review / stress-pentest | — | cross A06 / out | — | See §0 |
| O9 | Org-wide consistency | — | out of scope | — | See §0 |

## 2. Attack scenarios (in scope: 2/3, cross-ref: 1)

| # | A10 scenario template | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| S1 | A file-upload exception doesn't release resources → resource exhaustion (DoS) | ❌ | ✅ | `audit-security` (A10), `code-review`, `implement` | Resource cleanup on every path (finally/using/defer) |
| S3 | A multi-step financial transaction isn't rolled back on network interruption → account drain/double payment (race condition) | ❌ | ✅ | `audit-security` (A10), `code-review`, `implement`, `tdd` | Atomicity + idempotency + fail-closed |
| S2 | DB error shown in full to the user → SQLi reconnaissance | — | cross-ref A02 | — | See §0 |

## 3. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Preventions (9) | 5/9 in scope + 2 partial + 1 cross + 1 out | 0 | 5/5 (+ 2 partial) |
| Attack scenarios (3) | 2/3 in scope + 1 cross-ref | 0 | 2/2 |

Before: A10's items were not covered in the local skill set (A02's centralized error handling provided indirect grounding). After: 7/7 in-scope items are closed. Like A04/A05, A10 is a broad and highly code-visible category.

## 4. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A10 section**) | Swallowed/empty catch blocks, error handling only at the top level, missing resource cleanup on exceptional paths (unclosed file/connection/lock), non-atomic multi-step operations (no transaction/rollback, no idempotency), missing rate-limit/quota on expensive endpoints (resource exhaustion), fail-open on error. Cross-refs A02/A09/A01 | O1, O4, O5, S1, S3 |
| P0 | `code-review` | Security axis → ninth (final) sub-section **Exceptional Conditions (OWASP A10)** — gate: fires when the diff adds error handling, a multi-step/transactional operation, resource acquisition, or an expensive/unthrottled endpoint. Checks: (a) is the error caught at its origin and handled meaningfully (not swallowed)? (b) does it fail closed on error? (c) is the multi-step operation atomic (rollback on error), and idempotent if retryable? (d) are resources released on every path (including error)? (e) is the expensive operation throttled/quota'd? | O1, O2, O4, O5, S1, S3 |
| P1 | `implement` | New **"Exceptional conditions (OWASP A10)"** section: catch errors at their origin + handle meaningfully (never swallow silently); fail closed; make multi-step operations atomic (rollback on error) + idempotent if retryable; release resources on every path (including error) via finally/using/defer; throttle/quota expensive operations; route to a centralized error handler (A02: generic message to the user, A09: log it) | O1, O2, O3(partial), O4, O5, O6(partial), O7, S1, S3 |
| P1 | `tdd` | Item added to rules-of-the-loop: test exceptional/failure paths — the happy path isn't enough. For multi-step operations, verify **fail-closed** on mid-operation failure ("if the DB transaction crashes mid-way, the account isn't debited/there's no double payment"); verify graceful rejection under resource-exhaustion/limit conditions. Anti-pattern: an exceptional-condition-blind seam tested only on the happy path | O1, O4, S3 |

`to-spec`, `domain-modeling`, `improve-codebase-architecture` are untouched in this category — A06's misuse-case (abuse/failure scenario) already carries the design-time dimension; A10's code side is covered by implement/tdd/audit/code-review.

## 5. Two scopes: diff gate vs. project-wide scan (same pattern as A01-A05, A07-A09)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Exceptional Conditions sub-section) | diff (new/open) | every PR | `skills/code-review` | implemented |
| `audit-security` (A10 section) | whole project (existing/open) | pre-release / periodic | `skills/audit-security` | implemented |

## 6. Notes

- **Handoff from A09 completed:** the A09 analysis deliberately left exception-handling/fail-closed/rollback to A10; this document closes that handoff. A09 says the error **must be logged**; A10 says **how** the error should be handled (catch/fail-closed/rollback/cleanup) — the two complement each other.
- **A02 boundary:** the **existence** of a centralized error handler relates to both A02 (stack-trace leakage) and A10 (exceptional handling). A02 looks at "what's returned to the user" (generic message), A10 looks at "what happens internally" (fail-closed/rollback/cleanup) — same handler, two lenses.
- **tdd expansion:** this is the fifth security-test rule added to tdd (A01 denial, A05 injection, A06 misuse-case, A07 auth, A10 exceptional-path). All follow the same discipline: prove the risky/negative case before or alongside the happy path.
- **Top 10 completed:** with A01 (previous session) + A02-A10 (this session), the code-side coverage of OWASP Top 10:2025 has been worked into the skill set.

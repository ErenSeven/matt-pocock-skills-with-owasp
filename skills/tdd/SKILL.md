---
name: tdd
description: Test-driven development. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests.
---

# Test-Driven Development

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle — consult them before and during the loop, not after.

When exploring the codebase, read `CONTEXT.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

## What a good test is

Tests verify behavior through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification — "user can checkout with valid cart" tells you exactly what capability exists — and survives refactors because it doesn't care about internal structure.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Seams — where tests go

A **seam** is the public boundary you test at: the interface where you observe behavior without reaching inside. Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything — agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What's the public interface, and which seams should we test?"

When the shape of that interface is itself in question — how deep the module is, where the seam belongs, what the interface should expose — use the `/codebase-design` skill for the vocabulary. It is the shared source of the module, interface, depth, seam, adapter, leverage and locality terms, and it is a reference to consult, not a session to run.

**Choosing what kind of test a seam calls for** — unit, integration, component, contract, subcutaneous, or end-to-end — is a separate decision from picking the seam itself. See [test-types.md](test-types.md) for the definitions, the pyramid/sizes framing that governs how many of each you want, and the test-double taxonomy for the mocking choice that comes with it.

## Anti-patterns

- **Implementation-coupled** — mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behavior hasn't changed.
- **Tautological** — the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth — a known-good literal, a worked example, the spec.
- **Horizontal slicing** — writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead — one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first, then only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring is not part of the loop.** It belongs to the review stage (see the `code-review` skill), not the red → green implementation cycle.
- **Access-controlled seam → negative authz test first (OWASP A01).** When a seam exposes a protected resource or action, the first red test is the *denial* case: an unauthorized caller (missing/invalid credential) or a different owner (someone else's id) must be refused — 401/403 or an empty/blocked result — before you write the happy-path test. Ownership must be proven from the caller's identity, not a caller-supplied id; the test asserts you cannot reach another owner's record by passing its id. Add one such test per protected seam; treat a protected seam with only happy-path tests as an anti-pattern (see below).
- **Interpreter-facing seam → injection-payload test (OWASP A05).** When a seam feeds user input into an interpreter (SQL/NoSQL DB, OS/shell command, template engine, LDAP filter, expression evaluator, or HTML/JS output), add a test that sends a malicious payload — `' OR '1'='1`, `; cat /etc/passwd`, a `<script>` tag, an SSTI probe — and asserts it is treated as inert *data*, not executed: the query returns only the legitimately matching rows (not all), the command doesn't run, the markup comes back escaped. This mirrors the authz rule: prove the dangerous case is refused before trusting the happy path.
- **Derive misuse-case tests from the threat model (OWASP A06).** The two rules above are both *misuse-case tests* — a test written from the attacker's side, asserting the abuse is refused. When the spec has a **Threat Model / Abuse Cases** section, treat each abuse case there the same way: write a red test that performs the abuse (bypass a business-logic limit, cross a tenant boundary, drive a flow with bulk/automated actions) and asserts the control holds. A design control with no misuse-case test is unproven.
- **Authentication seams have their own misuse-cases (OWASP A07).** When a seam handles login, sessions, or tokens, add the auth-specific negative tests alongside the denial test: the **session id changes on login** (a fixed pre-login id must not survive → fixation), and **unknown-user and wrong-password return the same error** (and comparable timing → no enumeration). Expired/invalidated sessions and tokens with a bad signature/claim must be refused.
- **Test the exceptional path, not just the happy path (OWASP A10).** When a seam handles errors, performs a multi-step/stateful operation, or acquires resources, write a test that drives the *failure*: inject a mid-operation failure (a dependency throws, the network drops between step 2 and 3) and assert it **fails closed** and leaves **no partial state** — the classic being "if the transaction fails after the debit, the account is not left debited (no double-spend)." Assert resources are released and retries don't double-apply. A failure path with no test is where the money leaks.

## Anti-patterns (continued)

- **Authz-blind happy path** — a protected seam tested only for the authorized/own-resource case, so a regression that drops the authorization check or the ownership filter (IDOR/BOLA) stays green. The tell: no test fails if you delete the auth gate. Pair every protected seam with a denial test.
- **Injection-blind happy path** — an interpreter-facing seam tested only with clean input, so a regression that switches a parameterized query back to string concatenation (or bypasses output encoding) stays green. The tell: no test fails if you make the sink unsafe. Pair every interpreter-facing seam with an injection-payload test.
- **Exceptional-condition-blind seam** — an error-handling / multi-step / resource-acquiring seam tested only for success, so a regression that fails open or drops the rollback stays green. The tell: no test exercises the failure path. Pair it with an exceptional-path test that asserts fail-closed, no partial state, and resources released.

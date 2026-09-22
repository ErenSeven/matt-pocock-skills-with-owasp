# Test Strategy — Periodic / Project-Wide Disciplines · Skill Revision Document (project-independent)

> **Purpose:** Itemize the test disciplines that **don't fit** a red→green loop (Smoke/Sanity/
> Regression, Mutation Testing, Performance tests — Load/Stress/Soak/Spike) and derive which
> skill should carry them and how.
>
> **Why a separate document:** the types in [T01](T01-pyramid-and-test-types-revision.md) are
> written test-first, inside `tdd`'s red→green loop. The ones here **aren't written, they're run
> periodically** — not per PR, but pre-release/scheduled. That's why they all land not in `tdd`
> but in **`audit-test`**, the twin of `audit-security` (see [T01](T01-pyramid-and-test-types-revision.md) §4).
>
> **Scope rule:** project/technology independent — no tool names. **Applied** — the `audit-test`
> skill was created ([skills/audit-test/SKILL.md](../../skills/audit-test/SKILL.md)); every
> discipline here lives in its §3 "Periodic disciplines" section as a presence/health inventory.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · 📝 planned (not yet applied)

---

## 1. Smoke / Sanity / Regression

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| S1 | Smoke test (broad-shallow, after every build) | ❌ | ✅ | `audit-test` | `audit-test` §3.2 inventories whether a smoke suite exists (critical-path, after build/deploy) |
| S2 | Sanity test (narrow, after a small fix) | ❌ | ✅ | `audit-test` | `audit-test` §3.2 asks whether there's a way to verify a single change without the full suite |
| S3 | Regression test (deep, after a meaningful change) | ❌ | ✅ | `audit-test` ✅, `code-review` ✅ | `audit-test` §3.2 checks whether a regression suite exists and actually runs in CI; `code-review`'s Tests axis asks, on bug-fix diffs, "is there a regression test that fails without the fix and passes with it" |

## 2. Mutation Testing

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| M1 | Coverage ≠ effectiveness (Fowler) | ❌ | ✅ | `tdd` | `test-types.md` states "coverage is a tool, not a quality signal" + how mutation testing actually measures it, reachable from `SKILL.md` |
| M2 | Mutation score inventory | ❌ | ✅ | `audit-test` | `audit-test` §3.1 checks whether mutation-testing setup exists (config/CI/report) and, if so, flags low-scoring high-value code (critical flows) |

**Note:** mutation testing is too expensive to run per PR (it re-runs the whole suite many times) — so it lands only in `audit-test`, not `code-review`.

## 3. Performance Tests

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| P1 | Load testing | ❌ | ✅ | `audit-test` | Definition in `test-types.md`; `audit-test` §3.3 asks whether a scenario exists (absent on a revenue-critical path = a finding) |
| P2 | Stress testing | ❌ | ✅ | `audit-test` | `audit-test` §3.3 (breakpoint & recovery) |
| P3 | Soak (endurance) testing | ❌ | ✅ | `audit-test` | `audit-test` §3.3 (leaks/degradation over time) |
| P4 | Spike testing | ❌ | ✅ | `audit-test` | `audit-test` §3.3 (sudden burst) |

**Note:** performance tests, being project-wide/periodic by nature, also land in `audit-test`; `code-review`'s diff gate doesn't evaluate them (statically measuring a PR's performance impact is out of code-review's scope).

## 4. Contract Testing — provider side (additional note to T01)

[T01](T01-pyramid-and-test-types-revision.md) tied contract testing, as a "core type," to `tdd` (consumer side, seam selection). The provider side — "is the contract still being verified, when was it last verified" — is a periodic check, so it recurs here:

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| C1 | Provider contract-verification freshness | ❌ | 🟡 | `audit-test` | `audit-test` §2 flags missing contract tests at service boundaries; the "when was it last verified" freshness distinction is left open (presence is covered, freshness is a separate CI signal) |

---

## 5. Summary: coverage table (before → after)

| Heading | Before ✅ | Before ❌ | After (planned) | After (applied) |
|---|:--:|---|:--:|:--:|
| Smoke/Sanity/Regression (3) | 0 | S1–S3 | 3/3 | 3/3 ✅ (S3 regression: `audit-test` + `code-review`'s Tests axis) |
| Mutation (2) | 0 | M1–M2 | 2/2 | 2/2 ✅ |
| Performance (4) | 0 | P1–P4 | 4/4 | 4/4 ✅ |
| Contract — provider (1) | 0 | C1 | 1/1 | 🟡 (presence ✅, freshness open) |

---

## 6. Skill revisions (planned)

| Priority | Skill | Behavior added (planned) | Items closed |
|---|---|---|---|
| **P0** | `audit-test` (**new**, global) | ✅ **APPLIED** — §3 posture checklist: smoke/sanity/regression, mutation score, load/stress/soak/spike; §2 contract gaps | S1, S2, S3, M2, P1, P2, P3, P4, C1 |
| **P1** | `tdd` | ✅ **APPLIED** — the coverage-vs-mutation explanation written into `test-types.md` and reachable from `SKILL.md` | M1 |
| **P2** | `code-review` | ✅ **APPLIED** — the Tests axis includes a regression-test check on bug fixes (is there a test that fails without the fix) | S3 |

---

## 7. Skill-by-skill analysis

### 7.1 `audit-test` — ✅ APPLIED
- **Before:** No skill sweeps any of these disciplines project-wide. → **❌**.
- **After:** §3 "Periodic disciplines" written using `audit-security`'s "each section has its own posture checklist" model — mutation testing (§3.1), smoke/sanity/regression (§3.2), performance tests (§3.3), each with a present/absent + current? (✅/🟡/❌) inventory and a "most important missing discipline for this project's risk profile" finding. → **✅**.

### 7.2 `tdd` — ✅ APPLIED
- **Before:** The idea that coverage isn't a quality signal never existed in this skill set (even though it's in Fowler's own guide, it had never been carried over). → **❌**.
- **After:** Written into `test-types.md`, reachable via the `SKILL.md` pointer. → **✅** (see the same wiring state as [T01](T01-pyramid-and-test-types-revision.md) §5.1).

### 7.3 `code-review` — ✅ APPLIED
- **Before:** No check evaluates regression risk. → **❌**.
- **After:** The Tests axis (see [T01](T01-pyramid-and-test-types-revision.md) §5.3) includes a regression-test check on bug-fix diffs (item c: "is there a test that fails without the fix and passes with it"). The project-wide existence of a regression suite still lives in `audit-test`; the two are complementary. → **✅**.

### Summary

| Skill | Before | After | Status |
|---|:--:|:--:|---|
| `audit-test` | ❌ (absent) | ✅ | ✅ applied (global) |
| `tdd` | ❌ | ✅ | ✅ applied |
| `code-review` | ❌ | ✅ | ✅ applied (Tests axis, regression) |

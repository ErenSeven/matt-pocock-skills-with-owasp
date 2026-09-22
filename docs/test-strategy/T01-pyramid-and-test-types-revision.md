# Test Strategy — Pyramid & Test Types · Skill Revision Document (project-independent)

> **Purpose:** Itemize the test pyramid (Fowler) + Google's size model and the 8 core test types
> (Unit, Integration, Component, Contract, Subcutaneous, E2E, Acceptance/BDD, Snapshot), and for
> each item derive whether the **current skill set covers it** and which revision needs to be
> added to which skill.
>
> **Scope rule:** This document is not tied to any specific project or technology. "Covered" =
> *a skill's current instructions carry this test type as a behavior/decision point*. Tool names
> (a specific framework/library) are deliberately **never mentioned** — stack mapping is left to
> each project's own `.claude/skills/` override (see [USAGE.md](../USAGE.md) §8).
>
> **Source note:** Martin Fowler's testing guide, the Google Testing Blog, Microsoft Learn, the
> official Jest documentation. **Applied** — [tdd/test-types.md](../../skills/tdd/test-types.md)
> was written as the vocabulary; `tdd` (Seams pointer), `audit-test` (new skill), `code-review`
> (Tests axis), `to-spec` (Test levels), and `implement` (Test the right level) are all wired in.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · 📝 planned (not yet applied)
Columns: **Before** = state prior to this work · **After** = target state once the revision plan is applied

---

## 1. Framework: Pyramid and Size model

| # | Topic | Before | After | Revised skill | What the revision does/did |
|---|---|:--:|:--:|---|---|
| F1 | Test Pyramid (Fowler) — few e2e, many unit, integration in between | ❌ | ✅ | `tdd` | `test-types.md` §"Two frames" written + now reachable from `SKILL.md`'s Seams section |
| F2 | Google Test Sizes (Small/Medium/Large + constraints) | ❌ | ✅ | `tdd` | Same section of `test-types.md`, same access path |

**Before:** no skill carried a shared vocabulary for test types — every skill used its own undefined notion of "seam"/"test". **After:** `test-types.md` is the shared reference; `tdd` now points to it (✅), `audit-test`/`code-review`/`to-spec` not yet at this point (see §4).

---

## 2. Core test types

| # | Test type | Before | After | Revised skill | What the revision does/did |
|---|---|:--:|:--:|---|---|
| T1 | Unit | 🟡 | ✅ | `tdd` | `SKILL.md` already covered this implicitly ("test from the public interface"); `test-types.md` adds a formal definition + seam mapping, `SKILL.md` now provides access |
| T2 | Integration | ❌ | ✅ | `tdd` ✅, `audit-test` ✅ | Definition + "when do you touch real infrastructure" criterion in `test-types.md`; `audit-test` §1 (Pyramid balance) inventories the project-wide unit/integration ratio, §2 the hollow-middle gap |
| T3 | Component | ❌ | ✅ | `tdd` ✅, `audit-test` ✅ | Definition in `test-types.md`; `audit-test` §2 asks whether critical services are tested at their own level |
| T4 | Contract (consumer-driven) | ❌ | ✅ | `tdd` ✅, `audit-test` ✅ | Consumer/provider distinction in `test-types.md`; `audit-test` §2 flags missing contract tests at service boundaries (provider-side freshness: [T02](T02-periodic-disciplines-revision.md)) |
| T5 | Subcutaneous | 🟡 | ✅ | `tdd` | Already covered implicitly by `to-spec`'s "pick the highest seam" principle; now named in `test-types.md`, tied to the seam principle, reachable from `SKILL.md`; `audit-test` §2 asks about the subcutaneous/e2e alternative for a critical flow |
| T6 | End-to-end / broad-stack | ❌ | ✅ | `tdd` ✅, `audit-test` ✅, `to-spec` ✅, `code-review` ✅ | `audit-test` §1 ice-cream-cone + §2 critical-flow e2e gaps; `to-spec` "Test levels" makes the critical-flow → e2e/subcutaneous decision explicit at spec time; `code-review`'s Tests axis checks the level in the diff |
| T7 | Acceptance / BDD (Given-When-Then) | 🟡 | ✅ | `to-spec` ✅ | "User Stories" was already close to business language; `to-spec`'s "Testing Decisions" gained a Given-When-Then (Gherkin) option (when a stakeholder needs it) |
| T8 | Snapshot | ❌ | ✅ | `tdd` ✅, `audit-test` ✅, `code-review` ✅ | Definition + non-deterministic-value warning in `test-types.md`; `audit-test` §2 and `code-review`'s Tests axis check whether snapshots stub non-determinism and whether a change looks intentional; `to-spec` also notes it at spec time |

**Before:** **0** of the 8 types were covered directly, **2** (Unit, Subcutaneous, Acceptance) indirectly/weakly. **After:** `test-types.md` defines 8/8 and is wired into four skills — `tdd` (write-time), `audit-test` (project-wide), `to-spec` (spec-time), `code-review` (diff gate). **8/8 fully ✅.**

---

## 3. Summary: coverage table (before → after)

| Heading | Before ✅ | Before ❌/🟡 | After (planned) | After (applied) |
|---|:--:|---|:--:|:--:|
| Framework (2) | 0 | F1–F2 | 2/2 content written | 2/2 wired (`tdd` + `audit-test`) |
| Core test types (8) | 0 | T1–T8 (T1, T5, T7 weak) | 8/8 content written | **8/8 full wiring** (four skills: `tdd`, `audit-test`, `to-spec`, `code-review`) |

**Note:** "Content written" = the definition exists in `test-types.md`. "Wiring" = the relevant skill (`tdd/SKILL.md`, `audit-test`, `to-spec/SKILL.md`, `code-review/SKILL.md`) actually points to/uses it. All P0+P1 revisions have been applied — reaching the same level as the "after" state of the OWASP A01 work.

---

## 4. Skill revisions (planned, project-independent)

| Priority | Skill | Behavior added (planned) | Items closed |
|---|---|---|---|
| **P0** | `tdd` | ✅ **APPLIED** — a context pointer added to `SKILL.md`'s Seams section: when deciding on a seam/test type, check `test-types.md` | F1, F2, T1, T2, T3, T5 |
| **P0** | `audit-test` (**new**, global) | ✅ **APPLIED** — project-wide pyramid balance (§1) + layer/critical-flow coverage (§2) + periodic disciplines (§3); built on `audit-security`'s skeleton | T2, T3, T4, T6 |
| **P1** | `code-review` | ✅ **APPLIED** — a fourth axis, **Tests**, added (alongside Standards/Spec/Security): is the diff tested at the right level or at all, is there a regression test on a bug fix, are snapshots safe. With the "why a separate axis, not a Security sub-section" rationale | T6, T8 |
| **P1** | `to-spec` | ✅ **APPLIED** — "Testing Decisions" gained a "Test levels" sub-section (when to use integration/contract/e2e/subcutaneous/snapshot) and an optional Given-When-Then format | T4, T6, T7 |
| **P2** | `implement` | ✅ **APPLIED** — a "Test the right level" section added: cross-module→integration, service boundary→contract, critical flow→e2e/subcutaneous, UI→snapshot stub | T4 |

---

## 5. Skill-by-skill analysis: current (before) state and planned revision

### 5.1 `tdd` — ✅ APPLIED
- **Before:** Red→green loop, the seam concept, OWASP misuse-case rules. No distinction between test *types* (unit/integration/e2e/…) at all; pyramid/size model never mentioned. → **❌**.
- **After:** [test-types.md](../../skills/tdd/test-types.md) written as a new reference file (pyramid/sizes, 8 types, seam mapping). A single-paragraph context pointer added to `SKILL.md`'s "Seams — where tests go" section, right below the `/codebase-design` pointer: "Choosing what kind of test a seam calls for ... see test-types.md". → **✅**.

### 5.2 `audit-test` — ✅ APPLIED (new, global)
- **Before:** No skill performs a project-wide test-type/pyramid inventory (much like the absence of `audit-access-control` before A01). → **❌**.
- **After:** A new skill created carrying `audit-security`'s "per-section inventory + posture checklist + highest-risk finding" skeleton. Three sections: §1 Pyramid balance (level table + ice-cream-cone/hollow-middle/thin-base shape diagnosis), §2 Per-layer & critical-flow coverage (module/flow → test mapping, contract/snapshot gaps), §3 Periodic disciplines (mutation testing, smoke/sanity/regression, performance tests — presence/health inventory). References `test-types.md` as its vocabulary, technology-agnostic. → **✅**.

### 5.3 `code-review` — ✅ APPLIED
- **Before:** Three axes, Standards/Spec/Security; no axis evaluates test type/quality. → **❌**.
- **After:** A fourth **Tests** axis added — frontmatter/intro/Step 4 (self-contained sub-agent brief with the level vocabulary embedded) / Step 5 (`## Tests` heading) / "Why separate axes" (Tests example + overlap note with Security: security-negative tests stay under Security) / new "Why Tests is its own axis" section (rationale for a broad gate that conflicts with Security's narrow-gate model). → **✅**.

### 5.4 `to-spec` — ✅ APPLIED
- **Before:** Has a "Testing Decisions" section, but only requires a denial test + misuse-case test; no general test-type selection. → **🟡**.
- **After:** A **"Test levels"** sub-heading added — decides at spec time for integration (cross-module contract), contract (independently-deployed service), e2e/subcutaneous (critical flow), snapshot (UI); plus an optional **Given-When-Then** format note. References `test-types.md` as the vocabulary. → **✅**.

### 5.5 `implement` — ✅ APPLIED
- **Before:** Has A01–A10 "secure-by-default behavior" sections, nothing about test-type selection. → **❌**.
- **After:** A **"Test the right level"** section added right below the `/tdd` reference — follows the same "when the work touches X, do Y" pattern as the A01–A10 sections: cross-module→integration, independent service→contract, critical flow→e2e/subcutaneous, UI→snapshot stub; references `to-spec`'s Test levels decisions and the `test-types.md` vocabulary. → **✅**.

### Summary

| Skill | Before | After | Status |
|---|:--:|:--:|---|
| `tdd` | ❌ | ✅ | ✅ applied |
| `audit-test` | ❌ (absent) | ✅ | ✅ applied (global) |
| `code-review` | ❌ | ✅ | ✅ applied (Tests axis) |
| `to-spec` | 🟡 | ✅ | ✅ applied (Test levels) |
| `implement` | ❌ | ✅ | ✅ applied (Test the right level) |

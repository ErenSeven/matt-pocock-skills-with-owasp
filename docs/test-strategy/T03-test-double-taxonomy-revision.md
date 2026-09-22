# Test Strategy — Test Double Taxonomy · Skill Revision Document (project-independent)

> **Purpose:** Itemize Gerard Meszaros/Fowler's test-double classification (Dummy/Fake/Stub/Spy/
> Mock, state vs behavior verification, Classical/Detroit vs Mockist/London) and derive which
> revision needs to be layered on top of the existing [mocking.md](../../skills/tdd/mocking.md)
> ("mock at boundaries, SDK-style interface").
>
> **Why a separate document:** `mocking.md` already exists and works well — this document does
> **not delete** it, it adds a naming/taxonomy layer on top. Without the roles separated, "mock"
> was used as an umbrella term for all of them, which hid which verification style (state vs
> behavior) was actually in play.
>
> **Scope rule:** project/technology independent — no tool names (Moq/NSubstitute/Jest mock,
> etc.). **Applied** — [tdd/test-types.md](../../skills/tdd/test-types.md) content was written,
> [tdd/SKILL.md](../../skills/tdd/SKILL.md) points to it (P0), and
> [tdd/mocking.md](../../skills/tdd/mocking.md) now references it back (P1) — every item in this
> document's scope is reachable and discoverable.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · 📝 planned (not yet applied)

---

## 1. Test double roles

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| D1 | Dummy (fills a parameter slot, never used) | ❌ | ✅ | `tdd` | `test-types.md` §"Test doubles" — all 5 roles defined in one table, reachable from `SKILL.md` |
| D2 | Fake (a working but not production-suitable shortcut) | ❌ | ✅ | `tdd` | Same table |
| D3 | Stub (returns pre-programmed fixed answers) | 🟡 | ✅ | `tdd` | `mocking.md` already suggested stub-like usage ("mock at boundaries") but didn't name it; now named and reachable |
| D4 | Spy (stub + call recording) | ❌ | ✅ | `tdd` | New in `test-types.md` |
| D5 | Mock (expected-call + behavior verification) | 🟡 | ✅ | `tdd` | `mocking.md` used "mock" as an umbrella term; now separated into its narrow/correct meaning and reachable |

## 2. Verification style and TDD tradition

| # | Topic | Before | After | Revised skill | What was/will be done |
|---|---|:--:|:--:|---|---|
| V1 | State verification vs Behavior verification | ❌ | ✅ | `tdd` | Distinction in `test-types.md` + the "prefer state, survives refactors" rule — directly consistent with [SKILL.md](../../skills/tdd/SKILL.md)'s "implementation-coupled" anti-pattern |
| V2 | Classical (Detroit) TDD | 🟡 | ✅ | `tdd` | `mocking.md`'s "mock at system boundaries only, don't mock your own classes" principle was already close to Classical; now named and reachable |
| V3 | Mockist (London) TDD | ❌ | ✅ | `tdd` | Introduced in `test-types.md` as the opposing tradition — "outside-in, mock every collaborator with interesting behavior" |

---

## 3. Summary: coverage table (before → after)

| Heading | Before ✅ | Before ❌/🟡 | After (planned) | After (applied) |
|---|:--:|---|:--:|:--:|
| Double roles (5) | 0 | D1–D5 (D3, D5 weak) | 5/5 content written | 5/5 reachable |
| Verification style/tradition (3) | 0 | V1–V3 (V2 weak) | 3/3 content written | 3/3 reachable |

**Cross-link completed:** this section of `test-types.md` already referenced `mocking.md` ("see mocking.md"); `mocking.md` now also opens with a back-reference to `test-types.md` ("This file is the practical rule; that one is the vocabulary"). The two files point at each other, so either entry point leads to the other.

---

## 4. Skill revisions (planned)

| Priority | Skill | Behavior added (planned) | Items closed |
|---|---|---|---|
| **P0** | `tdd` | ✅ **APPLIED** — the single pointer added to `SKILL.md` (see [T01](T01-pyramid-and-test-types-revision.md) §4) already covers this section too, no separate pointer needed | D1–D5, V1–V3 (reachability) |
| **P1** | `tdd/mocking.md` | ✅ **APPLIED** — a back-reference added at the top of the file to `test-types.md`'s double-taxonomy section (cross-link, no content duplicated) | D3, D5, V2 (discoverability) |

A new skill is **not needed** — this topic stays entirely within `tdd`'s internal reference layer; neither `audit-test` nor `code-review` needs this taxonomy (a project-wide sweep or a diff gate doesn't care about test-double choice — that's purely a decision made while *writing* a test).

---

## 5. Skill-by-skill analysis

### 5.1 `tdd` (main file) — ✅ APPLIED
- **Before:** The test-double concept was never named; `mocking.md` gave a practical rule but no taxonomy/verification-style distinction. → **❌**.
- **After:** Added to `test-types.md`, with a one-way reference to `mocking.md`. The pointer in `SKILL.md`'s Seams section makes this section reachable too. → **✅**.

### 5.2 `tdd/mocking.md` — ✅ APPLIED
- **Before:** The "mock at system boundaries only" principle was already close to Classical/state-verification but didn't name it, and had no back-reference to `test-types.md`. → **🟡**.
- **After:** A back-reference added at the top of the file to `test-types.md`'s "Test doubles" section — without duplicating content, with the "this file is the practical rule, that one is the vocabulary" distinction. → **✅**.

### Summary

| Skill/file | Before | After | Status |
|---|:--:|:--:|---|
| `tdd/test-types.md` (doubles section) | ❌ (absent) | ✅ | ✅ applied, reachable |
| `tdd/SKILL.md` | ❌ (no pointer) | ✅ | ✅ applied |
| `tdd/mocking.md` | 🟡 | ✅ (with cross-link) | ✅ applied |

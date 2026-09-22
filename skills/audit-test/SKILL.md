---
name: audit-test
description: Audit a whole project's test suite — a coverage inventory to run before a release, when onboarding a codebase, or periodically. Scans the entire codebase (not a single diff) for where testing is thin or missing: pyramid balance (too few unit tests, or a slow top-heavy pile of e2e tests), modules and critical flows with no test at any level, missing test types for the surface that needs them (integration tests that never hit real infrastructure, service boundaries with no contract test, critical flows with no end-to-end test, flaky snapshots), and periodic disciplines a red-green loop can't show (mutation testing, smoke/sanity/regression suites, load/stress/soak/spike performance testing). Reports per-concern gaps ranked by risk. This inventories EXISTING tests — it does not write tests (use tdd), does not review one diff or PR (use code-review), and is not a security audit (use audit-security). Framework- and language-agnostic.
---

Project-wide **test audit** — the *inventory* companion to `code-review`, exactly as `audit-security` is to `code-review`'s Security axis:

- `code-review`'s **Tests axis** = **diff gate** — catches a change that ships new/changed behavior with no test, or at the wrong level.
- `audit-test` (this skill) = **whole-project sweep** — finds testing gaps that already exist across the codebase, regardless of when they crept in.

Run this periodically or before a release, not on every change. It reads the whole test suite and the code it covers, not a diff. Run whichever concern sections are relevant — all of them for a full audit, or a subset if the user asks for one concern specifically.

**Vocabulary:** this skill uses the test-type names, the pyramid/sizes framing, and the double taxonomy defined in [`../tdd/test-types.md`](../tdd/test-types.md). It is the single source of truth for what each term means; this skill *applies* those terms to a whole codebase rather than redefining them. Read it first if the distinctions (unit vs integration vs component vs subcutaneous vs e2e; smoke vs sanity vs regression; the four performance shapes) aren't fresh.

**Project-local tools:** if `docs/agents/test-tools.md` exists, read it first — it maps this repo's actual tools and commands (test runner, contract tool, mutation tool, load tool) onto the vocabulary above. Use it to make the posture checks concrete: a discipline the mapping lists is *expected*, so its absence in the code is a sharper finding; a blank row means the repo never claimed that tool, so note the discipline as simply not set up. Without the file, judge tool presence from the repo directly (config, CI steps, dependencies) and stay tool-agnostic.

**This skill reports, it does not write tests.** Fixes go through `implement` + `tdd` — the gap this audit names becomes the next red test.

## 1 — Pyramid balance

### 1. Inventory the suite by level

Sweep every test in the project and classify each by the level it actually exercises (not by which folder it sits in — a test in `unit/` that spins up a real DB is an integration test). Use the [`test-types.md`](../tdd/test-types.md) definitions. Count:

- **Unit** — isolated, no network/filesystem (Google "Small").
- **Integration** — real infrastructure, localhost only (Google "Medium").
- **End-to-end / broad-stack** — the whole system through its real front door (Google "Large").
- **Everything in between** — component, subcutaneous, contract — noted at the level they enter.

Where the runner reports it, capture rough runtime per level too — the pyramid is about speed as much as count.

### 2. Judge the shape

Compare the distribution against the pyramid ideal (many fast unit tests at the base, few slow e2e at the top):

- **Ice-cream cone** — a wide top: many slow/brittle e2e tests carrying confidence that cheaper, lower tests should carry. The tell: the suite is slow and flaky, and most coverage lives in e2e.
- **Hollow middle** — unit and e2e present but almost no integration, so cross-module contracts (repository↔DB, controller↔service) are only exercised end-to-end or not at all.
- **Thin base** — little unit coverage, so fast feedback is missing and every regression surfaces late in a slow test.

### 3. Report — pyramid inventory

**A. Level table** — count (and runtime where available) per level, with the shape named (healthy pyramid / ice-cream cone / hollow middle / thin base).

End with: the single most costly imbalance and the cheapest shift that fixes it (usually "push confidence down a level" — replace a cluster of e2e tests with integration or unit tests at the same seam).

## 2 — Per-layer & critical-flow coverage

### 1. Map modules and critical flows to their tests

Identify the project's modules/layers and its **critical flows** (the ones an outage would hurt most — login/auth, payment or money movement, data sync, anything the domain treats as core). For each, find the tests that actually cover it, at any level.

### 2. Flag the gaps

| Column | What to answer |
|---|---|
| **Covered at all?** | Does this module/flow have *any* test, or is it dark? |
| **Right level?** | Is a cross-module contract only covered by a slow e2e test (should have integration), or a critical flow only by unit tests of its parts (should have one end-to-end or subcutaneous test proving the whole path)? |
| **Service boundary?** | If the module talks to another independently-deployed service it doesn't control, is there a **contract test** — or is the integration only ever exercised by a full, brittle end-to-end run? |
| **UI drift?** | For rendered UI, are there snapshot tests, and do they stub non-deterministic values (timestamps, ids) so they aren't flaky-by-construction? |

### 3. Report — coverage inventory

**A. Coverage table** — one row per module/critical flow, most-critical-and-least-covered first, with the gap named (untested module / wrong-level coverage / missing contract test / snapshot with unstubbed non-determinism).

End with: count of critical flows with **no test at any level**, count of service boundaries with **no contract test**, and the single highest-risk gap (a payment flow with no end-to-end proof outranks a missing snapshot on an admin screen).

## 3 — Periodic disciplines

These don't get written test-first in a red-green loop — they run periodically against the whole suite or system. This section is a **presence/health inventory**: you check whether each exists and is current, you don't run it yourself.

### 1. Mutation testing — is test *effectiveness* measured?

Line coverage says a line *ran*; it says nothing about whether a test would notice that line breaking (see [`test-types.md`](../tdd/test-types.md)). Check:

- Is there any mutation-testing setup in the repo (config, a CI step, a stored mutation-score report)? Presence, not the tool.
- If a mutation score is available, which areas score low — covered but not actually verified? Flag high-value code (the critical flows from section 2) with low mutation score as the worst case: it looks tested and isn't.
- If there's no mutation testing at all, note that coverage numbers alone can't tell effective tests from execution-only ones.

### 2. Smoke / sanity / regression — do the intent-based suites exist?

Using the [`test-types.md`](../tdd/test-types.md) distinctions:

- **Smoke** — is there a broad-but-shallow critical-path suite that runs after each build/deploy?
- **Regression** — is there a suite meant to catch breakage of already-working behavior after a change/refactor, and is it actually run (in CI), not just present?
- **Sanity** — is there a lightweight way to check one recent change without the full suite?

Note absence, not just presence — a project with thousands of unit tests but no smoke suite still can't answer "is this deploy even alive?".

### 3. Performance testing — are the load shapes covered where they matter?

For systems where performance is a real requirement, check whether each shape exists as a scenario (see [`test-types.md`](../tdd/test-types.md) for the four):

- **Load** (expected peak), **Stress** (beyond capacity — breakpoint & recovery), **Soak** (sustained — leaks/degradation over time), **Spike** (sudden burst).
- Which are present, which absent. A revenue-critical or high-traffic path with **no** load test is the finding here; a purely internal batch tool with none is a judgement call, not a gap.

### 4. Report — periodic-discipline posture

**A. Posture checklist** — mutation testing, smoke, sanity, regression, and each performance shape as ✅ present-and-current / 🟡 present-but-stale-or-partial / ❌ absent, with a one-line evidence note each.

End with: the single most important missing discipline for *this* project's risk profile (a payment API with no load test and no regression suite outranks a CLI tool missing soak testing).

## Notes

- This skill **reports**, it does not write tests. A named gap becomes the next red test via `implement` + `tdd` (write the failing test at the missing level first).
- A finding here is a **judgement call** grounded in evidence — point at the module/flow and the tests that do (or don't) cover it. Don't assert a gap you can't point at.
- **Framework- and language-agnostic:** infer the test runner, levels, and CI setup from the repo (test folders, config, CI files, coverage/mutation reports) rather than assuming a specific stack. Tool names (a particular runner, mutation tool, load tool, or contract-test tool) are a **project-local** concern — this skill names *what kind* of test is missing; a project's own `.claude/skills/` or docs map that to its chosen tools (see the stack-mapping note in `docs/KULLANIM-REHBERI.md`).
- Overlap with `code-review`'s Tests axis is deliberate and mirrors `audit-security` ↔ `code-review` Security: this skill is the whole-project sweep of *existing* gaps; the Tests axis is the per-diff gate on *new* ones. The per-concern analysis behind each section lives in the revision docs under `docs/test-strategy/`.

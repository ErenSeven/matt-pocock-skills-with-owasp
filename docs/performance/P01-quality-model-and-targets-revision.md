# Performance — Quality Model, Metrics & Targets · Skill Revision Document (project-independent)

> **Purpose:** Itemize the shared language needed to turn performance into a measurable requirement
> (ISO/IEC 25010 Performance Efficiency), the metrics (SRE Four Golden Signals, percentiles) and the
> target-setting discipline (SLI/SLO, Azure WAF PE:01), and for each item derive **whether the
> current skill set covers it** and **which revision needs to be added to which skill**.
>
> **Why this is the first document:** Without a target, neither the diff gate (P02), nor the
> project-wide inventory (P03), nor the pass criterion of a perf test (P04) can be measured against
> anything. All of them rest on the answer to "what is acceptable for this flow?" — and that answer
> must be written at spec time.
>
> **Scope rule:** Project/technology-independent — tool names (APM, load-test tool, profiler)
> **never appear**. Concrete thresholds and tools stay in each project's own layer
> (see the [T04](../test-strategy/T04-project-local-tool-layer-revision.md) pattern).
>
> **Source note:** Primary sources read: Google SRE Book (*Monitoring Distributed Systems*,
> *Service Level Objectives*), Microsoft Azure Well-Architected Framework Performance Efficiency
> (checklist PE:01–PE:12, *performance targets*). The normative text of ISO/IEC 25010:2023 is paid
> (CHF 155, 22 pages, iso.org) and was not purchased — the sub-characteristic definitions rely on the
> iso25000.com summary; that is sufficient since the skills only use the names and short definitions. **Applied** —
> [performance-targets.md](../../skills/to-spec/performance-targets.md) was written as the vocabulary; `to-spec` (Performance Targets section + Testing Decisions), `tdd/test-types.md`, `audit-test` §3, `diagnosing-bugs` (Perf branch) and `implement` were wired up.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · 📝 planned (not yet applied)
Columns: **Before** = state prior to this work · **After** = target state once the revision plan is applied

---

## 0. Decisions taken

| # | Decision | Rationale | Effect |
|---|---|---|---|
| D1 | **No 5th axis is added** to `code-review`; performance checks run **gated** — as a `### Performance` sub-section inside the Standards axis, only when the diff touches a performance surface (query, I/O inside a loop, new endpoint, collection processing, cache, resource acquisition…) | Don't lengthen every PR; consistent with the Security axis's "skip if the gate doesn't match" model | The gate list and checks are written in **P02**; P01 only provides the vocabulary and the targets |
| D2 | The performance/security **boundary**: resource consumption *triggered by an attacker* (rate limit, payload/page limit, timeout — OWASP API4:2023, A10(e)) remains a security finding; *inefficiency under legitimate traffic* is a performance finding | So the same hunk isn't double-reported on two axes (code-review's existing "attribute to one" rule) | The P02/P03 gate definitions carry this distinction |

---

## 1. Framework: shared vocabulary (ISO/IEC 25010 Performance Efficiency)

| # | Topic | Before | After | Revised skill | What the revision does |
|---|---|:--:|:--:|---|---|
| F1 | **Time behaviour** — the degree to which response time and throughput meet requirements | ❌ | ✅ | new `performance-targets.md` | Definition + "latency / throughput / reaction time" distinction |
| F2 | **Resource utilization** — the degree to which the amount of resources used (CPU, memory, storage, network, connections) meets requirements | ❌ | ✅ | new `performance-targets.md` | Definition + the "resource budget" concept (e.g. queries per request, memory ceiling) |
| F3 | **Capacity** — the degree to which the upper limit of a parameter (concurrent users, requests/sec, data volume) meets requirements | ❌ | ✅ | new `performance-targets.md` | Definition + link to Load/Stress/Soak/Spike in `test-types.md` (these measure capacity) |

**Before:** No skill carries "performance" as a defined concept. `test-types.md` names four load shapes but doesn't say what they measure; `diagnosing-bugs` says "baseline measurement" but it's unclear what the baseline is of. **After:** The three sub-characteristics are defined in a single vocabulary file that plays for performance the role `test-types.md` plays for test types.

---

## 2. Metrics — what do we measure?

| # | Topic | Before | After | Revised skill | What the revision does |
|---|---|:--:|:--:|---|---|
| M1 | **Four Golden Signals** — latency, traffic, errors, saturation (SRE) | ❌ | ✅ | `performance-targets.md` | Definition of all four; the default SLI set for a user-facing system |
| M2 | **Percentiles, not averages** — p50 (typical), p95/p99 (tail); an average hides the tail (SRE example: 100 ms average, 1% at ~5 s) | ❌ | ✅ | `performance-targets.md`, `to-spec`, `diagnosing-bugs` | All targets and baselines are expressed as percentiles; an "average < X" target is flagged as a smell |
| M3 | **Successful and failed request latency kept separate** — fast errors skew the average, a slow error is worse | ❌ | ✅ | `performance-targets.md`, `diagnosing-bugs` | Distinction in the latency SLI definition; measurement kept separate in perf diagnosis |
| M4 | **Capacity metrics** — throughput (operations/time) and concurrency (concurrent users/operations) | ❌ | ✅ | `performance-targets.md`, `to-spec` | A target always states **under what load** it holds ("p95 < 300 ms @ 200 rps") |
| M5 | **Saturation & utilization target** — systems degrade before 100%; look at the most constrained resource, a rising p99 is an early signal | ❌ | ✅ | `performance-targets.md` | Tied to the resource budget (F2); the measurement/diagnosis method (USE/RED) is left to **P04** |

**Before:** **0 of** the 5 items are covered. `diagnosing-bugs` Phase 4 "Perf branch" gives a single tool example (`performance.now()`), no metric choice.

---

## 3. Target setting — the SLI / SLO discipline

| # | Topic | Before | After | Revised skill | What the revision does |
|---|---|:--:|:--:|---|---|
| H1 | **Numeric target** — no qualitative wording like "fast/slow"; a single value or a range (Azure PE:01) | ❌ | ✅ | `to-spec` | The new "Performance Targets" section requires a numeric target; doesn't accept qualitative wording |
| H2 | **Per-flow target, critical flows first** — user flow, business flow, data flow, external dependency (PE:01, PE:09) | 🟡 | ✅ | `to-spec` | The "critical flow" concept already exists on the test side (`to-spec` Test levels, `audit-test` §2); targets are tied to the same flow list |
| H3 | **The target derives from user need, not current performance** (SRE) | ❌ | ✅ | `to-spec` | Stated explicitly in the section instructions |
| H4 | **Few targets + a safety margin** — every SLO must be defensible in a prioritization discussion; the internal target is stricter than the published one | ❌ | ✅ | `to-spec`, `performance-targets.md` | "One or two targets per critical flow, not per story" rule |
| H5 | **Error budget** — 100% isn't realistic; the violation rate is a budget | ❌ | ✅ | `performance-targets.md` | Definition only (operational use is outside skill scope, see §5) |
| H6 | **The target is set early** — before deploy; for a live system without a target, it is derived from production data | ❌ | ✅ | `to-spec` ✅, `audit-performance` (P03) | New feature → spec time; existing system → P03's inventory reports "critical flow with no target" as a finding |
| H7 | **"No performance requirement" is said explicitly** — the section isn't left empty | ❌ | ✅ | `to-spec` | Same as the "say so explicitly rather than omitting" pattern in the Access Control / Sensitive Data sections |

**Before:** `to-spec`'s Access Control, Sensitive Data, and Threat Model sections get security written at spec time; there's no counterpart for performance. The only indirect coverage is H2 (the critical-flow concept already existing for tests).

---

## 4. Tying targets to tests and diagnosis

| # | Topic | Before | After | Revised skill | What the revision does |
|---|---|:--:|:--:|---|---|
| B1 | **A perf test's pass criterion = the SLO threshold** — "a load test exists" isn't enough; it must say which percentile threshold the test asserts at which load | 🟡 | ✅ | `tdd/test-types.md`, `to-spec`, `audit-test` | A "pass criterion" note under the `test-types.md` table; in `to-spec` Testing Decisions, a perf test + threshold for a flow with a target; in `audit-test` §3, **a load test asserting no threshold = 🟡** |
| B2 | **Diagnosis baseline against the target** — in a perf regression the baseline is taken as percentiles and compared to the target | 🟡 | ✅ | `diagnosing-bugs` | In the Perf branch: baseline as p50/p95/p99, successful/failed separate, compared with the spec target if any |

**Before:** `audit-test` §3 asks about the *presence* of load shapes, not what they measure (🟡). `diagnosing-bugs` says "measure first" but doesn't give the form of the measurement (🟡).

---

## 5. Deliberately out of scope

| Topic | Why it's out | Where it's handled |
|---|---|---|
| Capacity planning (PE:02), service/tier selection (PE:03), scaling/partitioning (PE:05) | Infrastructure/operations decision; not the behavior of code-and-spec skills | Out of scope (an ADR if needed — `domain-modeling`) |
| Live incident response, release decisions based on the error budget (PE:11, SRE) | Operations process | Out of scope |
| Code-level antipatterns (N+1, I/O in a loop, unindexed access — CISQ/ISO 5055, Azure antipatterns) | The diff gate's job | **P02** |
| Project-wide performance inventory | Periodic work | **P03** (`audit-performance`) |
| Measurement/diagnosis methods (USE, RED), perf test design (ISTQB CT-PT) | Diagnosis/test methodology | **P04** |
| Front-end metrics (Core Web Vitals: LCP/INP/CLS, p75 field data) | Web UI projects only | **P05** (optional) — the P01 vocabulary is extended |

---

## 6. Skill revisions (applied, project-independent)

| Priority | Skill | Behavior added | Items closed |
|---|---|---|---|
| **P0** | new `skills/to-spec/performance-targets.md` | ✅ **APPLIED** — Vocabulary file: the three ISO 25010 sub-characteristics, Four Golden Signals, the percentile rule, the successful/failed distinction, SLI/SLO/error budget, the "target + load condition" form (`p95 < X @ N rps`) | F1–F3, M1–M5, H5 |
| **P0** | `to-spec` | ✅ **APPLIED** — A **"## Performance Targets"** section between Threat Model and Testing Decisions: per critical flow an SLI + percentile target + load condition + resource budget (optional); derive from user need; few targets; say so explicitly if there's no requirement. In Testing Decisions: a perf test asserting the threshold for a flow with a target | H1–H4, H6, H7, B1 |
| **P1** | `tdd/test-types.md` | ✅ **APPLIED** — Under the Performance testing table: each shape's pass criterion is a target threshold (percentile @ load); a run without a threshold is a "measurement", not a test. Pointer to `performance-targets.md` | B1 |
| **P1** | `audit-test` | ✅ **APPLIED** — In §3: load test exists but asserts no threshold → 🟡; if a critical flow has no written target, report that as the root gap (the project-wide target inventory is in P03 `audit-performance`) | B1 |
| **P2** | `diagnosing-bugs` | ✅ **APPLIED** — In the Perf branch: baseline as percentiles (p50/p95/p99), successful/failed separate, compared with the spec target if any | M2, M3, B2 |
| **P2** | `implement` | ✅ **APPLIED** — A short "Performance targets" item: if the spec has a target, measure the critical flow against it; the perf test proving the target is in the spec's Testing Decisions | H1, B1 |
| — | `code-review` | **No change in P01.** The Spec axis already sees the spec's targets but can't verify them from the diff since that needs measurement; the gated Performance sub-section (D1) is in **P02** | — |

**Location note (vocabulary file):** Just as `test-types.md` sits under `tdd`, the performance vocabulary sits next to the skill that first applies it (`to-spec`); other skills point to it by relative path. **Copilot caveat:** `generate-copilot.ps1` only copies the `SKILL.md` body — helper `.md` files don't reach Copilot (the same existing limit applies to `test-types.md`). So the new section in `to-spec` must be written **self-sufficient** (the percentile + load-condition rule in the body), with the vocabulary file only adding depth.

---

## 7. Per-skill analysis: current (before) state and the planned revision

### 7.1 `to-spec` — ✅ APPLIED
- **Before:** Security is written at spec time via three sections (Access Control, Sensitive Data, Threat Model); test levels in Testing Decisions. Nothing for performance — not even a sentence like "should be fast" appears in the template. → **❌**.
- **After:** A "Performance Targets" section; for each critical flow `SLI · percentile target · load condition`, optional resource budget; the target's source is user need; explicit statement if there's no requirement. Testing Decisions ties targets to perf tests. → **✅**.

### 7.2 `performance-targets.md` (new) — ✅ APPLIED
- **Before:** Didn't exist. → **❌**.
- **After:** The performance counterpart of `test-types.md`: a single vocabulary, no tool names, P02–P05 all point to it. → **✅**.

### 7.3 `tdd/test-types.md` — ✅ APPLIED
- **Before:** Has a "which question does it answer" table for Load/Stress/Soak/Spike; no pass criterion. → **🟡**.
- **After:** Each shape's pass criterion is a target threshold. → **✅**.

### 7.4 `audit-test` — ✅ APPLIED
- **Before:** §3 inventories the presence of load shapes. → **🟡**.
- **After:** An existing-but-thresholdless load test is reported as 🟡. → **✅**.

### 7.5 `diagnosing-bugs` — ✅ APPLIED
- **Before:** "Establish a baseline, then bisect. Measure first, fix second." — correct but formless. → **🟡**.
- **After:** Baseline as percentiles, successful/failed separate, compared with the target. → **✅**.

### 7.6 `implement` — ✅ APPLIED
- **Before:** Has OWASP and test-level sections; no performance. → **❌**.
- **After:** An obligation to measure if the spec has a target (code-level rules to be added in P02). → **✅**.

### Summary

| Heading | Items | Before ✅ | Before 🟡 | Before ❌ | After |
|---|:--:|:--:|:--:|:--:|:--:|
| Framework (F) | 3 | 0 | 0 | 3 | 3/3 |
| Metrics (M) | 5 | 0 | 0 | 5 | 5/5 |
| Target setting (H) | 7 | 0 | 1 | 6 | 7/7 (the existing-system half of H6 is in P03) |
| Tying to tests/diagnosis (B) | 2 | 0 | 2 | 0 | 2/2 |
| **Total** | **17** | **0** | **3** | **14** | **17/17 ✅** |

---

## Sources

- Google SRE Book — [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/) (Four Golden Signals, tail latency, histogram)
- Google SRE Book — [Service Level Objectives](https://sre.google/sre-book/service-level-objectives/) (SLI/SLO/SLA, percentiles, choosing targets, error budget)
- Microsoft Azure Well-Architected — [Performance Efficiency checklist](https://learn.microsoft.com/azure/well-architected/performance-efficiency/checklist) (PE:01–PE:12)
- Microsoft Azure Well-Architected — [Define performance targets (PE:01)](https://learn.microsoft.com/azure/well-architected/performance-efficiency/performance-targets)
- ISO/IEC 25010:2023 Performance Efficiency — summary: [iso25000.com](https://iso25000.com/index.php/en/iso-25000-standards/iso-25010/59-performance-efficiency) (normative text is paid)
- OWASP API Security Top 10 2023 — [API4: Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/) (for the D2 boundary)

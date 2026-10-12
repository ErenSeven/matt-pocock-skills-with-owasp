# Performance — Measurement, Diagnosis & Perf Test Design · Skill Revision Document (project-independent)

> **Purpose:** P01–P03 established *what* is measured (target), *which patterns* are risky (baseline) and *where*
> they are searched for (audit). P04 closes the remaining two gaps: **how a slowdown is diagnosed** (USE, RED,
> dividing the time, anti-methodologies) and **how a meaningful perf test is set up** (operational profile, realistic
> data, environment, baseline, cadence).
>
> **Prerequisite:** [P01](P01-quality-model-and-targets-revision.md) (vocabulary, target = pass criterion) ·
> [P03](P03-project-wide-inventory-revision.md) (`audit-performance` §2 measurement posture).
>
> **Scope rule:** Project/technology-independent — no profiler, APM, or load-tool names.
>
> **Source note:** Primary sources read: Brendan Gregg — *The USE Method* and *Performance Analysis
> Methodology* (anti-methodologies + methodologies); Grafana Labs — *The RED Method* (Tom Wilkie);
> Microsoft Azure WAF — *PE:06 Performance testing*. The **ISTQB CT-PT** syllabus PDF was downloaded but its text
> couldn't be extracted (embedded font-subset glyphs; no poppler/pypdf in the environment — the same problem as the OWASP PDF).
> The ISTQB contribution rests on the **chapter headings** on the official certification page and the *operational
> profile* / *load testing* definitions in the ISTQB glossary; the syllabus text was not verified for detailed wording.
> **Applied** — see §4.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered
Columns: **Before** = state prior to this work · **After** = state once the revision is applied

---

## 1. Diagnosis — how is a slowdown narrowed down?

| # | Topic | Source | Before | After | Revised skill | What was done |
|---|---|---|:--:|:--:|---|---|
| D1 | **Problem statement & workload characterization** — what got slower, since when, what changed; is it the *load* or the *data volume* that changed? | Gregg | 🟡 | ✅ | `diagnosing-bugs` | Perf branch step 1; "twice the traffic / a table ten times bigger isn't a code regression" |
| D2 | **RED** — per service Rate, Errors, Duration distribution | Wilkie / Grafana | ❌ | ✅ | `diagnosing-bugs`, `performance-targets.md`, `audit-performance` | Perf branch step 2; "Two lenses" in the vocabulary; mapped to golden signals in audit §2 |
| D3 | **USE** — per resource Utilization, Saturation, Errors; software resources (pool, lock, queue, handle) included; errors first, non-zero saturation = a lead, 100% + saturation = bottleneck | Gregg | ❌ | ✅ | `diagnosing-bugs`, `performance-targets.md`, `audit-performance` | Perf branch step 3; vocabulary; in audit §2 "pool saturation is the most commonly missing signal" |
| D4 | **Dividing the time / drill-down** — split the slow request's duration into synchronous steps, drill into the largest; look at the slowest trace, not the average | Gregg, Azure (Chatty I/O) | ❌ | ✅ | `diagnosing-bugs` | Perf branch step 4 |
| D5 | **Scientific method / one variable** — falsifiable hypothesis, one change at a time | Gregg, PE:06 | ✅ | ✅ | `diagnosing-bugs` | Already present in Phase 3 + Phase 4; the perf-specific form was added: **baseline vs variant under the same load** (step 5) |
| D6 | **Anti-methodologies** — streetlight, random change, blame-someone-else | Gregg | ❌ | ✅ | `diagnosing-bugs` | An explicit warning in the Perf branch |
| D7 | **Data volume and load are kept when minimizing a perf repro** — N+1 is invisible on 10 rows | (Phase 2 ↔ perf) | ❌ | ✅ | `diagnosing-bugs` | Last sentence of the Perf branch (an exception to the Phase 2 minimise rule) |

**Before:** `diagnosing-bugs`'s general bug discipline (feedback loop, hypothesis, one variable) was strong (D5 ✅); after P01 the perf branch knew about the percentile baseline but didn't say *where to look*.

---

## 2. Perf test design — how is a meaningful test set up?

| # | Topic | Source | Before | After | Revised skill | What was done |
|---|---|---|:--:|:--:|---|---|
| T1 | **Test types** — load / stress / spike / soak; **capacity/scalability** = load increased step by step until the target stops holding | ISTQB ch.1, PE:06 | 🟡 | ✅ | `tdd/test-types.md` | Capacity definition added (tied to the ISO 25010 *capacity* sub-characteristic) |
| T2 | **Load first, then stress/spike/soak as risk warrants** | PE:06 | ❌ | ✅ | `tdd/test-types.md` | "not all at once" |
| T3 | **Operational profile** — the real task mix and proportions, average + peak, ramp-up, realistic pauses between steps | ISTQB glossary (*operational profile*), ch.1 *load generation* | ❌ | ✅ | `test-types.md`, `to-spec`, `audit-test` | test-types rule; a profile + data volume note in to-spec Testing Decisions; in audit-test "one endpoint in a loop" = 🟡 |
| T4 | **Realistic data** — production-like volume and variety; small data hides N+1/missing indexes, identical input flatters the cache | PE:06 | ❌ | ✅ | `test-types.md`, `to-spec`, `audit-test` | The same three places |
| T5 | **Environment** — as close to production as criticality warrants; a mocked dependency's latency isn't in the result | PE:06 | ❌ | ✅ | `test-types.md` | "Know what is mocked" |
| T6 | **Acceptance criterion = percentile threshold** | PE:06, P01 | ✅ | ✅ | — | Applied in P01 |
| T7 | **Performance budget per layer** — which layer when the test breaks | PE:06 | ❌ | ✅ | `performance-targets.md` | "Budgets per layer" (optional) |
| T8 | **Record a baseline + compare; drift; re-baseline on an architectural change** | PE:06, ISTQB ch.4 | ❌ | ✅ | `test-types.md`, `audit-test` | Rule + "no baseline" = 🟡 in the audit |
| T9 | **Cadence** — pipeline/scheduled, not one-off | PE:06 | ❌ | ✅ | `test-types.md`, `audit-test` | Same |
| T10 | **Proving an improvement** — before/after measurement under the same load | PE:06 (hypothesis-driven) | ❌ | ✅ | `implement` | Added to "Meet the performance targets" |

---

## 3. Deliberately out of scope

| Topic | Why |
|---|---|
| Controlled/progressive load testing in production | An operational decision and risk management; not the skills' behavior |
| Tool selection (ISTQB ch.5) | Project-local layer (`docs/agents/test-tools.md`, [T04](../test-strategy/T04-project-local-tool-layer-revision.md)) |
| Deep profiling techniques like Method R, off-CPU analysis, flame graphs | Tool/platform-dependent; the perf branch gives the direction with "drill into the largest" and leaves choosing the technique to the person diagnosing |
| Alert/health-model design | Lives outside the repo (same rationale as P03 §5) |

---

## 4. Skill revisions (applied)

| Skill | Behavior added | Closed |
|---|---|---|
| `diagnosing-bugs` | ✅ A five-step narrowing order in the Perf branch (problem statement & workload → RED → USE → dividing the time → baseline vs variant under the same load), an anti-methodology warning, keeping data volume/load in a perf repro | D1–D7 |
| `to-spec/performance-targets.md` | ✅ "Two lenses: RED for services, USE for resources"; "Budgets per layer" | D2, D3, T7 |
| `tdd/test-types.md` | ✅ Capacity test, test-type order, a five-item test setup (operational profile, realistic data, environment, baseline, cadence) | T1–T5, T8, T9 |
| `to-spec` | ✅ Testing Decisions → Performance test: operational profile + data volume note | T3, T4 |
| `audit-test` | ✅ A "is it meaningful and current?" check in §3 — profile, data volume, baseline, cadence; 🟡 if missing | T3, T4, T8, T9 |
| `audit-performance` | ✅ RED/USE mapping + pool saturation in the §2 golden signals item | D2, D3 |
| `implement` | ✅ A performance improvement is proven with a before/after measurement under the same load | T10 |

### Summary

| Heading | Items | Before ✅ | Before 🟡 | Before ❌ | After |
|---|:--:|:--:|:--:|:--:|:--:|
| Diagnosis (D) | 7 | 1 | 1 | 5 | 7/7 ✅ |
| Test design (T) | 10 | 1 | 1 | 8 | 10/10 ✅ |
| **Total** | **17** | **2** | **2** | **13** | **17/17 ✅** |

---

## Sources

- Brendan Gregg — [The USE Method](https://www.brendangregg.com/usemethod.html) · [Performance Analysis Methodology](https://www.brendangregg.com/methodology.html)
- Grafana Labs — [The RED Method: how to instrument your services](https://grafana.com/blog/the-red-method-how-to-instrument-your-services/) (Tom Wilkie)
- Microsoft Azure Well-Architected — [PE:06 Performance testing](https://learn.microsoft.com/azure/well-architected/performance-efficiency/performance-test)
- ISTQB — [Certified Tester Performance Testing (CT-PT)](https://istqb.org/certifications/certified-tester-performance-testing-ct-pt/) · [syllabus PDF (2018)](https://www.istqb.org/wp-content/uploads/2024/11/ISTQB-CT-PT_Syllabus_v1.0_2018.pdf) (text couldn't be extracted) · glossary: [operational profile](https://www.imbus.de/en/glossar/term/operational-profile), [load testing](https://www.imbus.de/en/glossar/term/load-testing)

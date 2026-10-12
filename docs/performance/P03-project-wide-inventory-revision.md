# Performance — Project-Wide Inventory (`audit-performance`) · Skill Revision Document (project-independent)

> **Purpose:** Gather the performance risk that is invisible at the diff gate or already exists —
> critical flows with no target/measurement, instrumentation gaps, patterns spread across the codebase,
> architectural antipatterns — into a new **`audit-performance`** skill that scans the whole project. The
> performance-side twin of `audit-security` and `audit-test`.
>
> **Prerequisite:** [P01](P01-quality-model-and-targets-revision.md) (vocabulary, target form) and
> [P02](P02-code-level-antipatterns-revision.md) (the 8-item performance baseline + the 7 items handed off to P03).
>
> **Scope rule:** Project/technology-independent — no APM, ORM, or load-tool names. The skill **reads code,
> it doesn't measure**: it doesn't run load tests, open a profiler, or query production.
>
> **Source note:** Primary sources read: Azure WAF *PE:04 Monitoring* and *PE:09 Prioritize critical
> flows*; Azure Architecture Center *Busy Database*, *Busy Front End*, *Monolithic Persistence*,
> *Noisy Neighbor* (in addition to *No Caching*, read in P02). The CISQ items were inherited from P02.
> **Applied** — [skills/audit-performance/SKILL.md](../../skills/audit-performance/SKILL.md) was created;
> `audit-test`, `audit-security`, `code-review` were tied in with boundary/redirect notes; a triggering eval
> set `evals/triggers/audit-performance.json` was added;
> [USAGE.md](../USAGE.md) §7.8 was written.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · ➡️ handed off elsewhere
Columns: **Before** = state prior to this work · **After** = state once the revision is applied

---

## 0. Design decisions

| # | Decision | Rationale |
|---|---|---|
| A1 | New skill, `audit-test` skeleton: per section *inventory → report table → "highest risk" line*, with a three-item final report at the end | The team is used to the output of the two audit skills; same reading order |
| A2 | The **critical flow** is the prioritization axis of every section (PE:09) — flows are rated High/Medium/Low in §1, every finding in §3–4 is tied to the flow it sits on | The same pattern is not the same risk in a payment flow and in an admin batch job |
| A3 | Load test **presence and threshold assertion** **stay** in `audit-test` §3; `audit-performance` doesn't redo it, it references its result | No double inventory; `audit-test` already does this job in its P01-updated form |
| A4 | The code-level baseline (B1–B8) is **repeated in short form** in the skill body, not just a pointer to `code-review` | The Copilot generator only copies the `SKILL.md` body — the skill must be self-sufficient; `audit-security` repeats `code-review`'s categories the same way |
| A5 | Description triggering boundary: load testing (`audit-test`), a single diff (`code-review`), a live slowdown (`diagnosing-bugs`) are explicitly excluded; 1006/1024 characters | The "hardened description" pattern in `audit-test`/`audit-security` |

---

## 1. Targets & critical flows (§1)

| # | Topic | Before | After | Source | What the skill has |
|---|---|:--:|:--:|---|---|
| E1 | Identifying critical flows and rating them High/Medium/Low | 🟡 | ✅ | PE:09 | §1.1 — same as `audit-test` §2's flow list + high-traffic / latency-sensitive / paid tier |
| E2 | A critical flow with no target in an existing system (the second half of P01 H6) | ❌ | ✅ | PE:01, SRE | §1.2 "Target written?" |
| E3 | The form of the target: percentile + load condition | ❌ | ✅ | P01 vocabulary | §1.2 "Well-formed?" — 🟡 if half is missing |
| E4 | Whether the target is measured and tested | 🟡 | ✅ | PE:04, PE:06 | §1.2 "Measured?" (tied to §2) · "Tested?" (result of `audit-test` §3) |

## 2. Measurement posture (§2)

| # | Topic | Before | After | Source | What the skill has |
|---|---|:--:|:--:|---|---|
| O1 | Keeping latency as a distribution (histogram/percentile) | ❌ | ✅ | PE:04, SRE | "Latency as a distribution" |
| O2 | Keeping successful/failed request duration separate | ❌ | ✅ | SRE | "Success vs failure split" |
| O3 | The four golden signals on critical flows | ❌ | ✅ | SRE | "The four golden signals on critical flows" |
| O4 | Duration per dependency call (retry/timeout included) | ❌ | ✅ | PE:04 | "Dependency timings" |
| O5 | Trace/correlation id across services | ❌ | ✅ | PE:04 | "Tracing across boundaries" |
| O6 | Per-tenant resource usage tag | ❌ | ✅ | Noisy Neighbor | "Per-tenant attribution" |

## 3. Code-level patterns — project-wide (§3)

| # | Topic | Before | After | What the skill has |
|---|---|:--:|:--:|---|
| K1–K8 | The P02 baseline (B1–B8): per-item I/O, over-fetching, unindexed access, per-request client, unreleased/unbounded resource, blocking I/O, unbounded retry, quadratic work — instances **that already exist in the codebase** | ❌ (diff gate only, P02) | ✅ | §3.1 eight items (with CWE/antipattern labels); §3.2 the report groups repeats into one line and asks for a **common root cause** (a single data-access helper = a single fix) |

## 4. Architectural antipatterns (§4) — handed off from P02

| # | P02 source | Topic | Before | After | What the skill has |
|---|---|---|:--:|:--:|---|
| M1 | Z7 | Busy Database — business logic/formatting in the database | ❌ | ✅ | "Busy Database" (except aggregates the DB is optimized for) |
| M2 | Z8 | Busy Front End — heavy work inside the request process (even on a background thread) | ❌ | ✅ | "Busy Front End" → a separate worker via a queue |
| M3 | Z9 | Monolithic Persistence — logs/telemetry/queue/blob in the business database | ❌ | ✅ | "Monolithic Persistence" |
| M4 | Z10 | Noisy Neighbor — no per-tenant quota/throttle | ❌ | ✅ | "Noisy Neighbor" (+ a note to `audit-security` for the attacker side) |
| M5 | Z6 | No Caching — frequently read, slowly changing data fetched from the source on every request | ❌ | ✅ | "Missing caching" (+ hit/miss visibility) |
| M6 | C17 / CWE-1057 | Data access spreading outside the data layer | ❌ | ✅ | "Scattered data access" → a candidate for `improve-codebase-architecture` |
| M7 | C15–C16 / CWE-1043, 1042 | Oversized aggregates, static state outside a singleton | ❌ | ✅ | "Oversized aggregates / shared static state" |

**All 7 items handed off from P02** (C15–C17, Z7–Z10) and the project-wide form of Z6 are met here as M1–M7.

---

## 5. Deliberately out of scope

| Topic | Why | Where |
|---|---|---|
| Running load tests, profiling, reading production telemetry | The skill reads code, it doesn't measure | Load test presence: `audit-test` §3 · measurement/diagnosis method: **P04** |
| Capacity planning, service/tier selection, scaling (PE:02/03/05), the impact of operational tasks (PE:10), incident response (PE:11) | Operations/infrastructure decision | Out of scope (an ADR if needed) |
| Correctness of dashboard/alert content | Lives outside the repo | Out of scope — the skill only checks whether instrumentation exists *in the code* |
| Front-end metrics (Core Web Vitals) | Web UI only | **P05** (optional) |

---

## 6. Skill revisions (applied)

| Skill | Behavior added |
|---|---|
| `audit-performance` (**new**, a real folder in the repo) | ✅ Four sections (§1 targets & critical flows · §2 measurement posture · §3 code-level patterns · §4 architectural antipatterns) + final report (the top three risks tied to critical flows) + Notes (sibling audit boundaries) |
| `audit-test` | ✅ `audit-performance` exclusion in the description; in §3, a redirect to `audit-performance` §1–2 for the "no target" finding |
| `audit-security` | ✅ A boundary item in Notes (abuse limits + error path → here; inefficiency under legitimate traffic → `audit-performance`). The description was **not changed** since it is at 1023/1024 |
| `code-review` | ✅ The "Why Performance is a Standards baseline" section names `audit-performance` as the project-wide counterpart |
| `evals/triggers/audit-performance.json` | ✅ 18 queries (9/9); near neighbors: `code-review`, `diagnosing-bugs`, `audit-test`, `audit-security`, `implement`, `to-spec`, `improve-codebase-architecture` |
| `docs/USAGE.md` | ✅ §7.8 Performance strategy, catalog row, flow map, typical flow, list of setup-free skills |

### Summary

| Section | Items | Before ✅ | Before 🟡 | Before ❌ | After |
|---|:--:|:--:|:--:|:--:|:--:|
| Targets & critical flows (E) | 4 | 0 | 2 | 2 | 4/4 ✅ |
| Measurement posture (O) | 6 | 0 | 0 | 6 | 6/6 ✅ |
| Code level, project-wide (K) | 8 | 0 | 0 | 8 | 8/8 ✅ |
| Architecture (M) | 7 | 0 | 0 | 7 | 7/7 ✅ |
| **Total** | **25** | **0** | **2** | **23** | **25/25 ✅** |

**Note — eval not run:** the triggering set was written but couldn't be run because headless `claude -p` is not
authenticated in this environment (see `evals/README.md` "Known blockers") — the same situation as the `audit-test` set.

---

## Sources

- Microsoft Azure Well-Architected — [PE:04 Monitoring](https://learn.microsoft.com/azure/well-architected/performance-efficiency/monitoring) · [PE:09 Prioritize critical flows](https://learn.microsoft.com/azure/well-architected/performance-efficiency/prioritize-critical-flows)
- Azure Architecture Center — [Busy Database](https://learn.microsoft.com/azure/architecture/antipatterns/busy-database/) · [Busy Front End](https://learn.microsoft.com/azure/architecture/antipatterns/busy-front-end/) · [Monolithic Persistence](https://learn.microsoft.com/azure/architecture/antipatterns/monolithic-persistence/) · [Noisy Neighbor](https://learn.microsoft.com/azure/architecture/antipatterns/noisy-neighbor/noisy-neighbor) · [No Caching](https://learn.microsoft.com/azure/architecture/antipatterns/no-caching/)
- Google SRE Book — [Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- CISQ — [Coding Rules (Performance Efficiency)](https://www.it-cisq.org/coding-rules) (via P02)

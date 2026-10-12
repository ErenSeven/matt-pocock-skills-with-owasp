---
applyTo: "**"
description: "Audit a whole project for performance risk — an inventory to run before a release, when onboarding, or periodically (not per-PR). Scans the whole codebase for critical flows with no written performance target (a percentile at a stated load) or no measurement behind it; instrumentation gaps (averages instead of percentiles, no tracing across service calls, no per-dependency timings); code-level patterns that degrade under load wherever they already exist (N+1 and per-item I/O in loops, over-fetching and unpaginated lists, unindexed queries, per-request clients, unreleased resources and unbounded caches, blocking I/O, unbounded retry, quadratic work); and architecture-level antipatterns (busy database, busy front end, monolithic persistence, noisy neighbor, missing caching), plus Core Web Vitals for web UIs. Reports gaps ranked by risk to critical flows. It does not run load tests (load-test coverage is audit-test), review one diff (code-review), or diagnose a live slowdown (diagnosing-bugs)."
---

<!-- Auto-generated from skills/audit-performance/SKILL.md by scripts/generate-copilot.ps1. Do not edit by hand. -->


Project-wide **performance audit** — the *inventory* companion to `code-review`'s performance baseline, exactly as `audit-security` is to its Security axis and `audit-test` to its Tests axis:

- `code-review`'s **performance baseline** (inside the Standards axis) = **diff gate** — catches a change that *introduces* a pattern that degrades under load.
- `audit-performance` (this skill) = **whole-project sweep** — finds performance risk that already exists across the codebase, regardless of when it crept in.

Run this periodically or before a release, not on every change. It reads the whole codebase, not a diff. Run whichever sections are relevant — all of them for a full audit, or a subset if the user asks for one concern specifically.

**Vocabulary:** this skill uses the target shape (indicator · percentile · load condition), the Four Golden Signals, and the percentiles-not-averages rule defined in [`../to-spec/performance-targets.md`](../to-spec/performance-targets.md). Read it first if those aren't fresh. The rule that matters most here: **an average is not a measurement of the tail**, and a target with no load condition is not a target.

**This skill reads code, it doesn't measure.** It doesn't run load tests, profilers, or queries against production — it inventories what the code and repo show: what is targeted, what is instrumented, and which patterns are present. A finding is a risk with evidence (a file, a hunk, a missing config), never a claim that something is slow. Whether load tests exist and assert targets is `audit-test` §3's job — reference its result rather than redoing it.

**This skill reports, it does not fix.** Fixes go through `implement` (its Performance defaults) — and a fixed N+1 or per-request-client problem gets a deterministic query/call-count test via `tdd` so it can't come back.

## 1 — Targets & critical flows

### 1. Identify the critical flows

List the flows whose degradation would hurt most — the same critical flows `audit-test` §2 uses: sign-in, payment or money movement, checkout, data sync, anything the domain treats as core — plus high-traffic and latency-sensitive flows (real-time over batch, paid tier over free). Rate each High / Medium / Low.

### 2. Find the target for each

Search specs, ADRs, docs, and config for a written performance target per flow:

| Column | What to answer |
|---|---|
| **Target written?** | Is there any numerical target for this flow — or only adjectives ("fast"), or nothing? |
| **Well-formed?** | Is it a **percentile** (p95/p99, not an average) **at a stated load** (`p95 < 300 ms @ 200 rps`)? A target missing either half is 🟡. |
| **Measured?** | Is there instrumentation (section 2) that could show whether the target is met in practice? |
| **Tested?** | Does a load scenario assert it (from `audit-test` §3, if run)? |

### 3. Report — targets inventory

**A. Flow table** — one row per critical flow, High-criticality-and-least-covered first, with the gap named (no target / qualitative target / average-based target / no load condition / unmeasured / untested).

End with: count of High-criticality flows with **no well-formed target**, and the single flow whose missing target is the highest risk. A flow with no target is the root gap — nothing downstream (tests, alerts, review) has anything to check against.

## 2 — Measurement posture

Judge from the code and repo config what the system *can* tell you about its own performance (Azure WAF PE:04). You are checking that the instrumentation exists and has the right shape, not reading dashboards.

### 1. Check the signals

- **Latency as a distribution** — are request/operation durations recorded as histograms or percentiles, or only as averages/counters? Averages hide the tail.
- **Success vs failure split** — are failed requests' durations recorded separately, so fast errors don't flatter the numbers?
- **The four golden signals on critical flows** — latency, traffic (request rate), errors, saturation (pool/queue/connection usage) for each High flow from section 1. Equivalently: **RED** per service (rate, errors, duration distribution) and **USE** per resource (utilization, saturation, errors) — including software resources like connection and thread pools, whose saturation (waiters, queue length) is the signal most often missing.
- **Dependency timings** — are calls to the database, other services, and third parties individually timed, including retries and timeouts?
- **Tracing across boundaries** — in a multi-service system, is a trace/correlation id propagated across service calls and queues so one request's time can be attributed per hop?
- **Per-tenant attribution** — in a multitenant system, is resource use tagged by tenant, so a noisy neighbor can be seen?

### 2. Report — measurement posture

**A. Posture checklist** — each signal above as ✅ present on critical flows / 🟡 partial (some flows, or averages only) / ❌ absent, with a one-line evidence note (the file, middleware, or config that shows it).

End with: the single blind spot that would most delay finding a production slowdown on a critical flow.

## 3 — Code-level patterns

Sweep the whole codebase for the same patterns `code-review`'s performance baseline gates on per diff (sources: CISQ / ISO 5055 Performance Efficiency, Azure performance antipatterns). Prioritize by where they sit: a pattern on a High-criticality flow from section 1 outranks the same pattern in an admin batch job.

### 1. Sweep for each pattern

- **Per-item I/O in a loop** — a loop issuing a query, request, or file operation per element, including ORM lazy-loading of children (N+1). *(CWE-1050, 1060, 1073; Chatty I/O)*
- **Over-fetching** — all columns/rows fetched then filtered, projected, or aggregated in memory; list endpoints and queries with no pagination over data that grows without bound. *(CWE-1049; Extraneous Fetching)*
- **Unindexed access** — frequent filters, sorts, and joins on large tables against columns with no index (check schema/migrations); write-heavy tables carrying an index per query. *(CWE-1067, 1094, 1089)*
- **Per-request client/connection** — clients designed to be shared (HTTP, DB, queue, cache) created per request; data access without pooling. *(CWE-1072; Improper Instantiation)*
- **Unreleased / unbounded resource** — resources not released on the normal path; caches or static collections with no expiry or size bound. *(CWE-404, 401, 772, 775, 1091)*
- **Blocking I/O on a request path** — synchronous I/O on request/event threads; blocking on async results. *(Synchronous I/O)*
- **Unbounded retry** — retry loops with no cap, no backoff, retrying permanent errors, ignoring `Retry-After`; hand-rolled retry beside an existing retry facility. *(Retry Storm)*
- **Quadratic work** — nested-loop matching of collections, sequential search in a loop, string concatenation in a loop on hot paths. *(CWE-1046, 1067)*

### 2. Report — pattern inventory

**A. Pattern table** — one row per occurrence cluster (pattern · location · the flow it sits on · criticality), critical-flow occurrences first. Group repeated instances of one pattern in one module into a single row with a count, rather than listing each.

End with: the pattern with the widest blast radius (most occurrences on High flows), and whether it shares a root cause — e.g. one data-access helper that every N+1 goes through is one fix, not twenty.

## 4 — Architecture-level antipatterns

Patterns a single diff can't show because they're about how load is distributed across the whole system (Azure performance antipatterns, plus the design-level CISQ weaknesses).

### 1. Check each

- **Busy Database** — business logic, formatting, or string/XML manipulation pushed into stored procedures, triggers, or queries, making the shared database a compute bottleneck. Aggregation the database is optimized for is fine — flag processing it isn't.
- **Busy Front End** — CPU- or resource-heavy work run inside the request-serving process (even on background threads) instead of being queued to a separate worker, starving request handling.
- **Monolithic Persistence** — data with very different access patterns in one store: logs, telemetry, queued messages, or large blobs/documents sharing the business database.
- **Noisy Neighbor** (multitenant only) — no per-tenant quota, throttle, or bound on expensive operations, so one tenant's load degrades the rest. A limit an *attacker* can bypass is also a security finding (OWASP API4) — note it there too.
- **Missing caching** — frequently read, slowly changing data (reference data, configuration, catalog) fetched from the source on every request with no cache; or caching present with no hit/miss visibility.
- **Scattered data access** — data access spread outside the project's data-access layer, so query shape can't be reviewed or optimized in one place. *(CWE-1057)* Candidate for `improve-codebase-architecture`.
- **Oversized aggregates / shared static state** — objects aggregating very large numbers of child elements, or mutable static state outside a deliberate singleton, that grow with data volume. *(CWE-1043, 1042)*

### 2. Report — architecture inventory

**A. Antipattern checklist** — each item above as ✅ not present / 🟡 present on non-critical paths / ❌ present on a critical flow, with evidence.

End with: the one structural change that would most raise capacity on the critical flows — these fixes are bigger than a PR, so frame it as a candidate for `improve-codebase-architecture` or an ADR, not a ticket.

## 5 — Web front end (Core Web Vitals)

Run only if the project serves a web UI. For a browser-facing flow, server latency is half the story — the user-facing targets are LCP ≤ 2.5 s, INP ≤ 200 ms, CLS ≤ 0.1 at the 75th percentile of real page loads, mobile and desktop separately (see [`performance-targets.md`](../to-spec/performance-targets.md)).

### 1. Check measurement

- **Field data** — does the front end report LCP, INP, and CLS from real users (real-user monitoring), segmented mobile/desktop, aggregated at p75? Without it, nobody knows whether the targets are met.
- **Lab check** — does the pipeline run a page-load check on key pages with thresholds on LCP, CLS, and Total Blocking Time (the lab stand-in for INP)? A lab check without thresholds is a measurement, not a gate.

### 2. Sweep the key pages for patterns

- **LCP** — the main above-the-fold image lazy-loaded, injected by script, or only a CSS background; render-blocking synchronous scripts or large stylesheets in the head; the main content waiting on client-side rendering (large gap between first byte and first paint); slow server response for the HTML.
- **CLS** — images, videos, embeds, or ad slots with no reserved dimensions; banners or late content inserted above what's shown; animations of layout properties instead of `transform`; web fonts whose fallback is a very different size.
- **INP** — long synchronous work in event handlers or on load; very large DOM trees; heavy work that could move off the main thread.

### 3. Report — web vitals inventory

**A. Page table** — one row per key page (tied to its critical flow): field data ✅/❌, lab gate ✅/🟡/❌, and the LCP/CLS/INP patterns found with evidence.

End with: the page whose metric is most at risk on mobile, and the pattern behind it.

## Final report

After the sections run, end with **the three highest-risk findings across the audit**, each tied to a critical flow: what it is, the evidence, and why it outranks the rest. A High-criticality flow with no target and no measurement usually ranks first — every other gap on it is invisible until production.

## Notes

- A finding here is a **judgement call** grounded in evidence — point at the file, query, config, or missing instrumentation. Don't assert a risk you can't point at, and never claim something "is slow" from code alone.
- **Framework- and language-agnostic:** infer the data-access layer, clients, retry facility, instrumentation, and schema from the repo. Tool and library names (an APM, an ORM, a load tool) are a **project-local** concern; this skill names *what kind* of gap exists.
- **Boundaries with sibling audits:** load-test presence and whether tests assert targets → `audit-test` §3. Resource limits that stop **abuse** (rate limits, max page/payload size, timeouts) and resources not released on **error paths** → `audit-security` (A10). Inefficiency under legitimate traffic → here. Attribute each finding once.
- Overlap with `code-review`'s performance baseline is deliberate: this skill is the whole-project sweep of *existing* risk; the baseline is the per-diff gate on *new* risk. The per-item analysis behind each section lives in the revision docs under `docs/performance/`.

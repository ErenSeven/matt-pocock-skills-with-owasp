# Performance targets — shared vocabulary

What "performance" means when a skill asks for it, and the shape a target must take. This is the performance counterpart of `tdd`'s `test-types.md`: one vocabulary, referenced by `to-spec` (where targets are written), `implement`, `tdd`, `audit-test`, and `diagnosing-bugs`. Tool-agnostic — it names *what* to measure, never a monitoring, profiling, or load-test tool.

## Three things performance means (ISO/IEC 25010 Performance Efficiency)

| Sub-characteristic | Question it answers | Typical measure |
|---|---|---|
| **Time behaviour** | Are response time and throughput good enough while the system does its job? | Latency percentiles, requests/transactions per second |
| **Resource utilization** | Does it use an acceptable amount of CPU, memory, storage, network, connections? | Resource budget — e.g. DB queries per request, memory ceiling, payload size |
| **Capacity** | How far can a parameter go (concurrent users, request rate, data volume) before requirements stop being met? | The load at which a time-behaviour target still holds — measured by the Load/Stress/Soak/Spike shapes in `test-types.md` |

## What to measure — the Four Golden Signals

For any user-facing service, these are the default indicators (Google SRE):

- **Latency** — time to serve a request. Measure **successful and failed requests separately**: a fast error flatters the numbers, and a slow error is worse than a fast one.
- **Traffic** — demand on the system, in a unit that fits it (requests/sec, transactions/sec, concurrent sessions).
- **Errors** — rate of failed requests, including implicit failures (wrong content with a success status) and policy failures (any request slower than its target).
- **Saturation** — how full the most constrained resource is. Systems degrade before 100% utilization; a rising p99 is often the earliest sign.

### Two lenses: RED for services, USE for resources

The golden signals split into two checklists, used together when measuring or diagnosing:

- **RED** (per service — what users feel): request **R**ate, **E**rrors, **D**uration as a distribution.
- **USE** (per resource — CPU, memory, disk, network, and software resources such as connection/thread pools, locks, queues, file handles): **U**tilization (time busy), **S**aturation (work waiting — any non-zero queue is a lead), **E**rrors.

RED says *which* service is slow; USE says *which* resource behind it is the bottleneck.

## Percentiles, not averages

Express every latency target and every baseline as a **percentile** — p50 for the typical experience, p95/p99 for the tail. An average hides the tail: a 100 ms mean can contain 1% of requests taking seconds. A target written as "average < X" is a smell; rewrite it as a percentile.

## The shape of a target

A target is **numerical and conditional**: an indicator, a percentile threshold, and the load it holds under.

```
<flow>: <indicator> <percentile> < <threshold> @ <load condition>
checkout: latency p95 < 300 ms @ 200 rps sustained
search:   latency p99 < 1 s   @ 50 concurrent users
import:   throughput >= 10k rows/s for a 1M-row file
```

- **No qualitative words** — "fast", "responsive", "scalable" are not targets.
- **Per flow, critical flows first** — the same critical flows the spec's Test levels name (login, payment, data sync). One or two targets per critical flow, not one per story; each should be worth defending when priorities clash.
- **From user need, not current performance** — don't adopt a number just because the system meets it today.
- **Leave a margin** — an internal target stricter than any published commitment.

### Budgets per layer

A flow target can be split into **performance budgets** per layer — e.g. checkout p95 < 400 ms = API handler 150 ms + DB queries 150 ms + payment call 100 ms, with errors < 1%. Budgets aren't required, but when a test fails they say *which layer* blew it instead of only *that* the target was missed.

## SLI, SLO, error budget

- **SLI (indicator)** — the measured quantity (p95 checkout latency).
- **SLO (objective)** — the target value for it (< 300 ms).
- **SLA (agreement)** — an SLO with contractual consequences; most specs only need SLOs.
- **Error budget** — 100% compliance is unrealistic; the SLO implies a tolerated rate of misses (e.g. 99% of requests within target leaves 1% budget).

## Web front end: Core Web Vitals

For a flow a user experiences in a **web browser**, server latency isn't the whole story — the user-facing targets are the Core Web Vitals, judged at the **75th percentile of real page loads (field data)**, mobile and desktop segmented separately:

| Metric | Measures | Good | Poor |
|---|---|---|---|
| **LCP** — Largest Contentful Paint | Loading: when the largest image/text block in the viewport renders | ≤ 2.5 s | > 4.0 s |
| **INP** — Interaction to Next Paint | Responsiveness: click/tap/key → next frame, worst interactions over the visit | ≤ 200 ms | > 500 ms |
| **CLS** — Cumulative Layout Shift | Visual stability: the largest burst of unexpected layout shifts | ≤ 0.1 | > 0.25 |

```
product page: LCP p75 <= 2.5 s, INP p75 <= 200 ms, CLS p75 <= 0.1 (field, mobile)
```

- **Field vs lab.** Field data (real-user monitoring) is the judge; lab runs (a scripted page load in CI) catch regressions before release but can't replace it — and a lab run has no real user input, so it can't measure INP (Total Blocking Time is the lab proxy).
- **Diagnostics, not targets:** TTFB (server response) and FCP (first paint) explain a slow LCP. LCP splits into TTFB (~40%), resource load delay (<10%), resource load duration (~40%), and element render delay (<10%); INP splits into input delay, processing time, and presentation delay.

## Targets become test pass criteria

A performance test's pass criterion **is** a target: it asserts the percentile threshold at the stated load. A load run that asserts nothing is a measurement, not a test — useful, but it can't go red. A regression baseline is likewise taken in percentiles and compared against the target.

## Boundary with security

Resource consumption an **attacker** can trigger — missing rate limits, unbounded page/payload size, missing timeouts — is a security finding (OWASP API4:2023; `code-review`'s Exceptional Conditions sub-section). **Inefficiency under legitimate traffic** is a performance finding. Attribute each finding to one side, not both.

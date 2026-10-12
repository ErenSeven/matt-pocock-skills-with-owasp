# Performance — Front End: Core Web Vitals · Skill Revision Document (project-independent)

> **Purpose:** P01–P04 established server-side performance (response time, throughput, resources). For a flow
> experienced in the browser, these are half the story: the loading, interactivity, and visual stability the user feels
> are measured by the **Core Web Vitals** (LCP, INP, CLS). P05 adds these three metrics to the same layers — vocabulary,
> spec, write time, diff gate, diagnosis, test, project-wide inventory — **in a way that kicks in only for projects with a
> web UI**.
>
> **Prerequisite:** [P01](P01-quality-model-and-targets-revision.md) (target form, percentiles) ·
> [P02](P02-code-level-antipatterns-revision.md) (gated baseline) · [P03](P03-project-wide-inventory-revision.md) ·
> [P04](P04-measurement-diagnosis-and-test-design-revision.md) (dividing the time, lab vs real measurement).
>
> **Scope rule:** Project/technology-independent — no measurement library, lab tool, or framework names.
>
> **Source note:** Primary sources read (web.dev): *Web Vitals*, *Largest Contentful Paint*,
> *Optimize LCP*, *Interaction to Next Paint*, *Cumulative Layout Shift*, *Defining the Core Web Vitals
> metrics thresholds*. **Applied** — see §5.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered
Columns: **Before** = state prior to this work · **After** = state once the revision is applied

---

## 0. Design decisions

| # | Decision | Rationale |
|---|---|---|
| W1 | Every new item is **conditional**: only if there is a web UI ("if users experience the flow in a browser" in the spec, the "rendered web UI" gate at the diff gate, "Run only if the project serves a web UI" in the audit) | Some of the team's projects are backend/mobile; don't produce noise in an unrelated project |
| W2 | **A new item, not a new axis** in `code-review`: the baseline's 9th item "Web vitals regression", with its own gate (markup/styles/client script) | Consistent with P02 D1/G1; doesn't widen the gate of the other 8 items |
| W3 | Target = **the p75 of field data, mobile and desktop separately** | web.dev thresholds article: p75 covers most visits but is less affected by extreme network conditions |
| W4 | Lab measurement is **a regression gate, not proof of the target**; INP can't be measured in the lab → TBT as a proxy | web.dev *Web Vitals*: the lab contains no real user input |

---

## 1. Metrics & thresholds

| # | Topic | Before | After | Revised skill | What was done |
|---|---|:--:|:--:|---|---|
| V1 | **LCP** — render time of the largest visible content element; good ≤ 2.5 s, poor > 4.0 s | ❌ | ✅ | `performance-targets.md`, `to-spec` | Table in the vocabulary; Web UI target item in the spec |
| V2 | **INP** — click/tap/key → next frame; the worst interactions over the visit; good ≤ 200 ms, poor > 500 ms | ❌ | ✅ | same | same |
| V3 | **CLS** — the largest session window of unexpected shifts; good ≤ 0.1, poor > 0.25 | ❌ | ✅ | same | same |
| V4 | **p75, field data, mobile/desktop separately** | ❌ | ✅ | same | Target form example: `LCP p75 <= 2.5 s (field, mobile)` |
| V5 | **Diagnostic metrics** — TTFB, FCP (not targets); LCP's four sub-parts (TTFB ~40%, resource load delay <10%, download ~40%, render delay <10%); INP's three phases | ❌ | ✅ | `performance-targets.md`, `diagnosing-bugs` | "Diagnostics, not targets" in the vocabulary; dividing the time in diagnosis |

## 2. Code-level patterns (diff gate + write time + inventory)

| # | Pattern | Metric | Before | After | Where |
|---|---|:--:|:--:|:--:|---|
| W-1 | The main (above-the-fold) image is lazy-loaded, injected by script, or only a CSS background → the browser discovers it late | LCP | ❌ | ✅ | `code-review` B9, `implement`, `audit-performance` §5 |
| W-2 | A synchronous script / large render-blocking style in `<head>` | LCP | ❌ | ✅ | same |
| W-3 | The main content waiting on client-side rendering (a large gap between first byte and first paint) | LCP | ❌ | ✅ | `audit-performance` §5 (architectural; rarely visible in a single diff) |
| W-4 | Image/video/embed/ad slot with no reserved dimensions | CLS | ❌ | ✅ | `code-review` B9, `implement`, `audit-performance` §5 |
| W-5 | Content inserted late above existing content (banner etc.) | CLS | ❌ | ✅ | same |
| W-6 | Animating layout properties (`top`, `width`…) — instead of `transform` | CLS | ❌ | ✅ | same |
| W-7 | A web font whose fallback is a very different size | CLS | ❌ | ✅ | `audit-performance` §5 |
| W-8 | Long synchronous work in an event handler; heavy work that could be moved off the main thread | INP | ❌ | ✅ | `code-review` B9, `implement`, `audit-performance` §5 |
| W-9 | A very large DOM tree | INP | ❌ | ✅ | `audit-performance` §5 |

## 3. Measurement, test & diagnosis

| # | Topic | Before | After | Revised skill | What was done |
|---|---|:--:|:--:|---|---|
| M1 | Is there real-user measurement (RUM), mobile/desktop separately, p75 | ❌ | ✅ | `audit-performance` §5.1 | "Field data" check |
| M2 | A lab page-load check with thresholds in CI (LCP, CLS, TBT) | ❌ | ✅ | `tdd/test-types.md`, `audit-performance` §5.1 | The web counterpart in test-types; a thresholdless lab = 🟡 in the audit |
| M3 | Diagnosing a web slowdown: start from field data, reproduce in the lab on a throttled device profile, split the metric into its sub-parts | ❌ | ✅ | `diagnosing-bugs` | A "Web front end" paragraph in the Perf branch |

---

## 4. Deliberately out of scope

| Topic | Why |
|---|---|
| **Native mobile app performance** (startup time, dropped frames, ANR) | Core Web Vitals are web-specific; React Native/native apps have their own metrics — a separate P06 topic if needed |
| SEO / search ranking impact | Not a performance requirement, a product/marketing decision |
| Choice of measurement library, lab tool, CDN/image service | Project-local layer ([T04](../test-strategy/T04-project-local-tool-layer-revision.md)) |
| Experimental/pending web vitals metrics | web.dev lifecycle: only **stable** metrics become targets |

---

## 5. Skill revisions (applied)

| Skill | Behavior added | Closed |
|---|---|---|
| `to-spec/performance-targets.md` | ✅ "Web front end: Core Web Vitals" — threshold table, p75/field/segment rule, target example, lab vs field, diagnostic metrics and sub-parts | V1–V5, W4 |
| `to-spec` | ✅ A "Web UI" item in Performance Targets | V1–V4 |
| `code-review` | ✅ A 9th baseline item **Web vitals regression** (LCP/CLS/INP patterns) + "rendered web UI (opens only this item)" in the gate | W-1, W-2, W-4–W-6, W-8 |
| `implement` | ✅ "Web UI: protect the Core Web Vitals" in Performance defaults | W-1, W-2, W-4–W-6, W-8 |
| `tdd/test-types.md` | ✅ A thresholded lab check for the web; only field p75 proves the target | M2 |
| `diagnosing-bugs` | ✅ "Web front end" in the Perf branch — field → lab, LCP/INP/CLS dividing the time | M3, V5 |
| `audit-performance` | ✅ A new **§5 Web front end** (measurement, page scan, page table); "plus Core Web Vitals for web UIs" in the description (1008/1024 after shortening) | M1, M2, W-1–W-9 |
| `evals/triggers/audit-performance.json` | ✅ +2 queries (web inventory → triggers; a single page's LCP regression → `diagnosing-bugs`, doesn't trigger) | — |

### Summary

| Heading | Items | Before ✅ | Before ❌ | After |
|---|:--:|:--:|:--:|:--:|
| Metrics & thresholds (V) | 5 | 0 | 5 | 5/5 ✅ |
| Code-level patterns (W) | 9 | 0 | 9 | 9/9 ✅ |
| Measurement, test & diagnosis (M) | 3 | 0 | 3 | 3/3 ✅ |
| **Total** | **17** | **0** | **17** | **17/17 ✅** |

This document closes the performance series (P01–P05).

---

## Sources

- web.dev — [Web Vitals](https://web.dev/articles/vitals) · [Largest Contentful Paint](https://web.dev/articles/lcp) · [Optimize LCP](https://web.dev/articles/optimize-lcp) · [Interaction to Next Paint](https://web.dev/articles/inp) · [Cumulative Layout Shift](https://web.dev/articles/cls) · [Defining the Core Web Vitals thresholds](https://web.dev/articles/defining-core-web-vitals-thresholds)

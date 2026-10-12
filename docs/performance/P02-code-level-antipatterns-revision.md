# Performance — Code-Level Antipatterns & the Diff Gate · Skill Revision Document (project-independent)

> **Purpose:** Itemize the performance weaknesses visible in a diff from two primary catalogs — **CISQ / ISO/IEC 5055
> Performance Efficiency** (18 CWEs) and **Azure Architecture Center performance antipatterns** (10 items) —
> and for each, derive whether it's handled **at the diff gate (`code-review`), at write time (`implement`), or
> in the project-wide inventory (P03 `audit-performance`)**.
>
> **Prerequisite:** [P01](P01-quality-model-and-targets-revision.md) — the vocabulary ([performance-targets.md](../../skills/to-spec/performance-targets.md))
> and decisions D1 (gated, inside Standards) and D2 (security boundary). This document makes D1 concrete.
>
> **Scope rule:** Project/technology-independent. The examples in the sources come from a single ecosystem (ORM, HTTP
> client classes) — **no tool/class names are carried** into the skill text, only the pattern ("a new client per
> request", "a query inside a loop").
>
> **Source note:** Primary sources read: the CISQ *Coding Rules* page (Performance Efficiency
> section — 15 parent + 3 child CWEs), the Azure Architecture Center antipattern catalog + the Chatty I/O,
> Extraneous Fetching, Improper Instantiation, Synchronous I/O, Retry Storm, No Caching detail pages.
> The CISQ PDF couldn't be downloaded (connection reset); the list was taken from the web page carrying the same content.
> **Applied** — `code-review` (Step 3 performance baseline + gate, Standards brief, `### Performance` heading, "Why Performance is a Standards baseline" section, frontmatter), `implement` ("Performance defaults") and `tdd/test-types.md` (deterministic perf proxies) were updated.

Status legend: ✅ covered (applied) · 🟡 indirect/weak · ❌ not covered · 📝 planned (not yet applied) · ➡️ handed off to another document
Columns: **Before** = state prior to this work · **After** = target state once the revision plan is applied

---

## 0. Gate design (D1 made concrete)

| # | Decision | Rationale |
|---|---|---|
| G1 | Performance checks are added to `code-review`'s **Standards** sub-agent as a **"performance baseline"** — just like the existing *smell baseline*: the repo standard always overrides, every finding is a **judgement call** | No new sub-agent, no new axis; Standards already carries "a baseline that applies even if the repo documents nothing" |
| G2 | The baseline runs **gated**: if the diff touches none of the surfaces below, the Standards report says one line, "no performance-relevant surface" | `code-review`'s own principle: narrow gates become sub-sections, idle cost is zero ("Why Security stays one axis") |
| G3 | **Gate surface:** data access (query, ORM call, migration/index), **a loop containing I/O or a collaborator call**, creating a new outbound client/connection, retry policy, adding/changing a cache, collection processing on a request path, resource acquisition (file/connection/handle) | The narrowest set covering every diff-visible item in the catalogs |
| G4 | Report heading: **`### Performance`** under `## Standards` | So performance findings don't mix with style findings, without opening a new top-level heading either |
| G5 | **No measurement claim:** don't say "slow" from a diff; say "this pattern leads to X under load". If the spec has a target (P01), tie the finding to it | The sources' shared caveat: performance can only be proven by measuring under load — the diff gate only catches patterns |

**Attribution rules (no double reporting):**
- A resource not released **on the error path** → Security / Exceptional Conditions (A10(d)) · never released **on the normal path** / grows unbounded → Performance.
- No **client-controlled** limit (page size, payload, rate limit, timeout) → Security (A10(e), OWASP API4) · an **unbounded result set** under legitimate traffic → Performance (D2).
- Retry: a retry that **overwhelms its own system/the other side** when a dependency is down → Performance (Retry Storm) · a retry that is **not idempotent** (double processing) → Security / A10(c).

---

## 1. CISQ / ISO/IEC 5055 Performance Efficiency — 18 CWEs

| # | CWE | Weakness (summary) | Before | After | Where | What the revision does |
|---|---|---|:--:|:--:|---|---|
| C1 | **1050** | Excessive platform resource consumption within a loop (query/I/O/open-close in a loop) | ❌ | ✅ | `code-review`, `implement` | Baseline item **"Per-item I/O in a loop"** (merged with Chatty I/O, see Z1) |
| C2 | **1060** | Excessive number of inefficient server-side data accesses | ❌ | ✅ | `code-review`, `implement` | Same item (N+1 pattern) |
| C3 | **1073** | Non-SQL invokable control element with excessive number of data resource accesses | ❌ | ✅ | `code-review` | Same item (many separate queries in one function/handler) |
| C4 | **1049** | Excessive data query operations in a large data table | ❌ | ✅ | `code-review` | Baseline item **"Over-fetching"** (see Z2) |
| C5 | **1067** | Excessive execution of sequential searches of a data resource | ❌ | ✅ | `code-review` | Baseline item **"Unindexed access"** — does a new query filter/sort go to an unindexed field |
| C6 | **1094** | Excessive index range scan | ❌ | ✅ | `code-review` | Same item (index decision in the migration) |
| C7 | **1089** | Large data table with excessive number of indices | ❌ | ✅ | `code-review` | Same item, opposite direction: a new index per query on a write-heavy table |
| C8 | **1072** | Data resource access without use of a connection pool | ❌ | ✅ | `code-review`, `implement` | Baseline item **"Per-request client/connection"** (merged with Improper Instantiation, see Z3) |
| C9 | **404** | Improper resource shutdown or release (parent) | 🟡 | ✅ | `code-review`, `implement` | Baseline item **"Unreleased / unbounded resource"**; A10(d) only covered the error path → 🟡 |
| C10 | 401 (child) | Memory leak | ❌ | ✅ | `code-review` | Same item (including an unboundedly growing static collection/cache) |
| C11 | 772 (child) | Missing release of resource after effective lifetime | 🟡 | ✅ | `code-review` | Same item |
| C12 | 775 (child) | Missing release of file descriptor/handle | 🟡 | ✅ | `code-review` | Same item |
| C13 | **1091** | Use of object without invoking destructor/dispose | ❌ | ✅ | `code-review` | Same item (a disposable object not disposed) |
| C14 | **1046** | Creation of immutable text using string concatenation in a loop | ❌ | ✅ | `code-review` | A low-priority sub-example within baseline item **"Quadratic work"** |
| C15 | **1043** | Data element aggregating an excessively large number of non-primitive elements | ❌ | ➡️ | P03 | Design level; rarely visible in a single diff → project-wide inventory |
| C16 | **1042** | Static member data element outside of a singleton | ❌ | ➡️ | P03 | Design level; the "unboundedly growing static collection" case is caught in C10 |
| C17 | **1057** | Data access operations outside of the expected data manager component | ❌ | ➡️ | P03, `improve-codebase-architecture` | Architecture (scattered data access → one seam); not the diff gate's job |
| C18 | **424** | Improper protection of alternate path | ❌ | — | out of scope | CISQ lists this item under PE but its nature is access control — already covered under A01; **not taken** into the performance baseline |

**Before:** **0 of** the 18 items are covered; **3** (404, 772, 775) are **partially** covered (🟡) by A10(d)'s "release on the error path too" check. **After:** 14 items are gathered into 6 baseline items at the diff gate (B1–B5, B8), 3 are handed off to P03, 1 is deliberately out of scope.

---

## 2. Azure performance antipatterns — 10 items

| # | Antipattern | Visible in a diff? | Before | After | Where | What the revision does |
|---|---|:--:|:--:|:--:|---|---|
| Z1 | **Chatty I/O** — many small requests for one logical operation (N+1, endpoint per property, small file writes) | ✅ | ❌ | ✅ | `code-review`, `implement` | **"Per-item I/O in a loop"** — fix: combine into one query/join/batch; including the ORM's hidden lazy loading |
| Z2 | **Extraneous Fetching** — more data than needed (all columns, filter/aggregate in memory, unpaginated list) | ✅ | ❌ | ✅ | `code-review`, `implement` | **"Over-fetching"** — projection/filter/aggregate in the data layer; pagination for an unbounded list |
| Z3 | **Improper Instantiation** — creating and destroying per request an object designed to be shared (HTTP/DB/queue client) | ✅ | ❌ | ✅ | `code-review`, `implement` | **"Per-request client/connection"** — a single instance or a pool if shareable + thread-safe; watch for the race of setting properties per request on a shared object |
| Z4 | **Synchronous I/O** — blocking the calling thread while waiting on I/O | ✅ | ❌ | ✅ | `code-review`, `implement` | **"Blocking I/O on a request path"** — use the async API if there is one; synchronous waiting on async (sync-over-async) smell |
| Z5 | **Retry Storm** — unbounded/un-spaced retry | ✅ | ❌ | ✅ | `code-review`, `implement` | **"Unbounded retry"** — attempt/duration limit, growing delay (backoff), no retry on 4xx, honor `Retry-After`; use the existing retry/circuit-breaker infrastructure |
| Z6 | **No Caching** — fetching the same expensive data again and again | 🟡 | ❌ | ✅ | `code-review` (weak signal), P03 | Only as a question when the diff adds a **read of slowly-changing data on a hot path**; if a cache **is added**: does it have a lifetime/limit (if not, C10) |
| Z7 | **Busy Database** — offloading excessive processing to the data store | 🟡 | ❌ | ➡️ | P03 | A single procedure is visible in a diff, but "excessive" is understood from the project-wide load distribution |
| Z8 | **Busy Front End** — doing resource-intensive work in the front-end process (even on a background thread) | 🟡 | ❌ | ➡️ | P03 | Same rationale; if the heavy work in the diff is on the request path, it's raised under Z4/Quadratic |
| Z9 | **Monolithic Persistence** — keeping data with very different access patterns in a single store | ❌ | ❌ | ➡️ | P03 | Architecture |
| Z10 | **Noisy Neighbor** — one tenant's disproportionate resource use | ❌ | ❌ | ➡️ | P03 (+ the security side: API4 per-tenant quota) | Architecture/operations |

**Before:** **0 of** the 10 items are covered. **After:** the 6 visible in a diff (Z1–Z6) enter the baseline, 4 are handed off to P03.

---

## 3. Unified performance baseline (added to code-review Step 3)

The two catalogs reduce to 8 baseline items (B8 is the in-application form of CWE-1046/1067; not a separate heading in the sources). Each item is in *what* → *how to fix* form, in the same tone as the smell baseline:

| # | Baseline item | Source mapping | Gate surface |
|---|---|---|---|
| B1 | **Per-item I/O in a loop** — the loop issues a query/request/file operation per element (including ORM lazy loading). → gather into one query/batch/join | CWE-1050, 1060, 1073 · Chatty I/O | loop + data access |
| B2 | **Over-fetching** — fetching all columns/rows and then filtering, projecting, or aggregating in memory; an unbounded unpaginated list. → push filter/projection/aggregate into the data layer, paginate the list | CWE-1049 · Extraneous Fetching | data access |
| B3 | **Unindexed access** — a new query filter/sort/join goes to an unindexed field on a large table; or an index per query is added to a write-heavy table. → ship the index with the migration / consolidate indexes | CWE-1067, 1094, 1089 | data access, migration |
| B4 | **Per-request client/connection** — a client/connection designed to be shared is created per request; a connection without a pool. → one shared instance or a pool (verify it's thread-safe) | CWE-1072 · Improper Instantiation | new outbound client |
| B5 | **Unreleased / unbounded resource** — a resource not closed/disposed on the normal path; a cache or static collection growing with no expiry/limit. → scope-bound release; a lifetime + size limit on the cache | CWE-404, 401, 772, 775, 1091 · No Caching (cache side) | resource acquisition, cache |
| B6 | **Blocking I/O on a request path** — synchronous I/O on a request/event thread; synchronous waiting on async. → use the async API | Synchronous I/O | data access, outbound client |
| B7 | **Unbounded retry** — unbounded or un-spaced retry, retry on a permanent error (4xx). → attempt/duration limit, growing delay, honor `Retry-After`, use the existing retry/circuit-breaker infrastructure | Retry Storm | retry policy |
| B8 | **Quadratic work** — matching collections with nested loops on a request path / sequential search in a loop / string concatenation in a loop. → a lookup structure (map/set), a builder | CWE-1046, 1067 | collection processing |


**Tying to the target (P01):** If the spec has Performance Targets, a baseline finding touching a critical flow is reported with a reference to that target ("there's a checkout p95 target; this loop issues a query per cart item"). If there's no target, the finding still stands as a judgement call.

---

## 4. Skill revisions (applied, project-independent)

| Priority | Skill | Behavior added | Items closed |
|---|---|---|---|
| **P0** | `code-review` | ✅ **APPLIED** — In Step 3, the **performance baseline** (B1–B8, right below the smell baseline, with the same two rules: repo overrides · judgement call) + the G3 gate. In the Standards sub-agent brief, gated application of the baseline and the `### Performance` sub-heading. `### Performance` in the Step 5 aggregate. An attribution note in "Why separate axes" (A10(d)/(e) ↔ B5/B2, A10(c) ↔ B7). Frontmatter: "+ gated performance baseline" in the Standards definition | C1–C14, Z1–Z6 |
| **P1** | `implement` | ✅ **APPLIED** — A **"Performance defaults"** section (the write-time counterpart): no I/O in a loop (batch), fetch only the needed data + paginate unbounded lists, an index for a new query filter, reuse shareable clients, release resources by scope, a lifetime+limit on caches, no blocking I/O on a request path, bounded retry + backoff. Plus: **a query-count test for an N+1 regression** (an assertion at the data-access seam that "this operation issues a constant number of queries for N records") | C1–C2, C8, C9, Z1–Z5 |
| **P2** | `tdd/test-types.md` | ✅ **APPLIED** — **Deterministic proxy assertions** that catch a performance regression at unit/integration level (query count, call count, number of connections allocated) — perf tests that don't measure timing and aren't flaky in CI. They don't replace load tests; they cheaply lock in an N+1/instantiation regression | B1, B4 |
| — | P03 `audit-performance` | ➡️ C15–C17, Z6 (project-wide), Z7–Z10 will enter this skill's inventory | — |

**Why `to-spec` isn't touched in P02:** the spec-time decision (targets) was added in P01; antipatterns are write/review-time work.

---

## 5. Per-skill analysis: current (before) state and the planned revision

### 5.1 `code-review` — ✅ APPLIED
- **Before:** The Standards axis carries documented standards + a 12-item Fowler smell baseline; no performance patterns. Security/A10(d) asks about releasing resources on the error path, A10(e) about the quota of an expensive operation — these are the only touch points with performance. → **❌** (🟡 for resource release).
- **After:** A gated **performance baseline** (B1–B8) next to the smell baseline; one line if the gate doesn't match; a `### Performance` sub-heading; attribution rules. No new sub-agent/axis. → **✅**.

### 5.2 `implement` — ✅ APPLIED
- **Before:** P01 added "Meet the performance targets" (measure if there's a target); no code-level default behavior. → **🟡**.
- **After:** "Performance defaults" — in the "when the work touches X, do Y" pattern of the OWASP sections. → **✅**.

### 5.3 `tdd/test-types.md` — ✅ APPLIED
- **Before:** Perf tests are only load shapes (periodic); no deterministic per-diff perf assertion is defined. → **❌**.
- **After:** Query/call-count assertions are defined as "perf regression proxies". → **✅**.

### Summary

| Catalog | Items | Before ✅ | Before 🟡 | Before ❌ | After: diff gate | After: handed to P03 | Out of scope |
|---|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
| CISQ 5055 PE (C) | 18 | 0 | 3 | 15 | 14 | 3 | 1 |
| Azure antipatterns (Z) | 10 | 0 | 0 | 10 | 6 | 4 | 0 |
| **Total** | **28** | **0** | **3** | **25** | **20** | **7** | **1** |

---

## Sources

- CISQ — [Coding Rules (Performance Efficiency section)](https://www.it-cisq.org/coding-rules) · [Performance Efficiency measure](https://www.it-cisq.org/standards/code-quality-standards-performance-efficiency) · [Performance-Efficiency-Weaknesses.pdf](https://blog.it-cisq.org/pdf/Performance-Efficiency-Weaknesses.pdf)
- Azure Architecture Center — [Performance antipatterns catalog](https://learn.microsoft.com/azure/architecture/antipatterns/)
  - [Chatty I/O](https://learn.microsoft.com/azure/architecture/antipatterns/chatty-io/) · [Extraneous Fetching](https://learn.microsoft.com/azure/architecture/antipatterns/extraneous-fetching/) · [Improper Instantiation](https://learn.microsoft.com/azure/architecture/antipatterns/improper-instantiation/) · [Synchronous I/O](https://learn.microsoft.com/azure/architecture/antipatterns/synchronous-io/) · [Retry Storm](https://learn.microsoft.com/azure/architecture/antipatterns/retry-storm/) · [No Caching](https://learn.microsoft.com/azure/architecture/antipatterns/no-caching/)
- MITRE CWE — for each item `https://cwe.mitre.org/data/definitions/<id>.html`

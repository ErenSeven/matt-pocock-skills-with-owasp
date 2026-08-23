# OWASP A01:2021 — Broken Access Control · Skill Revision Document (project-independent)

> **Purpose:** Itemize A01 under three headings (common vulnerability types · preventions · attack scenarios),
> and for each item, derive **whether the current skill set covers it** and
> **which revision needs to be added to which skill**.
>
> **Scope rule:** This document is not tied to any specific project. "Covered" =
> *a skill's current instructions address this A01 item as a behavior*.
> Revisions must be written as **global skills** (`~/.claude/skills`) so they work across every project.
>
> **Source note:** The `owasp top 10.pdf` text could not be extracted (font-subset; no python/poppler in the environment).
> Based on the canonical **OWASP A01:2021** definition. **This is an analysis/planning document — the skill files had not yet been edited at the time of writing.**

Status legend: ✅ covered · 🟡 indirect/weak · ❌ not covered
Columns: **Before** = current skill state · **After** = state once the revision is applied

---

## 1. Common vulnerability types

| # | A01 vulnerability type | Before | After | Revised skill | What the revision does (what earns the checkmark) |
|---|---|:--:|:--:|---|---|
| Z1 | Least-privilege violation; "allow by default" (no deny-by-default) | ❌ | ✅ | `code-review`, `improve-codebase-architecture` | `improve-codebase-architecture` gets a "scattered authorization → single deny-by-default authz seam" deepening opportunity; `code-review`'s Access Control axis asks on every diff "is this fail-closed / deny-by-default" |
| Z2 | Access control bypassed by altering URL/parameter/HTML/API request (forced browsing, parameter tampering) | ❌ | ✅ | `code-review`, `implement` | `code-review`'s sub-agent adds a check for "does ownership come from a token, or from a client param"; `implement` gets a "don't take ownership from client id/param/body, take it from the token" rule |
| Z3 | IDOR/BOLA — accessing someone else's record by supplying its unique id | ❌ | ✅ | `tdd`, `code-review` | `tdd` gets a red test "cannot access with another owner's id" + an "authz-blind happy path" anti-pattern; `code-review` checks every endpoint for "is it scoped to the caller's own resource" |
| Z4 | Missing function-level access control on POST/PUT/DELETE | ❌ | ✅ | `code-review`, `to-spec` | `code-review`'s Access Control axis asks "is there an authz gate" (all verbs); `to-spec`'s Access Control section has the spec record allowed/forbidden actions per actor at spec time |
| Z5 | Privilege escalation (anonymous→user, user→admin) | ❌ | ✅ | `domain-modeling`, `code-review` | `domain-modeling` models "roles/privilege levels" as an invariant; `code-review`'s sub-agent asks "is there a privilege-escalation path (anon→user, user→admin)" |
| Z6 | Metadata manipulation (JWT/cookie/hidden-field replay/tampering) | 🟡 | ✅ | `security-review` (built-in) + `code-review` | Already covered by the built-in `security-review`; `code-review`'s Access Control axis adds an inline gate by verifying the token/session source on every diff |
| Z7 | CORS misconfiguration (unauthorized origin access) | ❌ | ✅ | `code-review` | The Access Control sub-agent brief adds a "flag any CORS fallback that falls through to AllowAnyOrigin" item |
| Z8 | Unauthenticated user forcing access to an authenticated page / regular user forcing access to a privileged page | ❌ | ✅ | `code-review`, `to-spec` | `code-review` checks "auth gate + fail-closed"; `to-spec` adds a "default posture = deny unless granted" + privilege-levels template |

**Before:** **none** of the A01 vulnerability types were directly covered by the local skill set
(`code-review` was Standards+Spec axes only). **After:** the P0+P1 revisions close **8/8**.

---

## 2. Preventions

| # | A01 prevention | Before | After | Revised skill | What the revision does (what earns the checkmark) |
|---|---|:--:|:--:|---|---|
| O1 | **Deny by default** except public | ❌ | ✅ | `improve-codebase-architecture` | Deepening opportunity: "a single authz seam = deny-by-default" pattern; `to-spec`/`implement` also carry "default posture = deny unless granted" |
| O2 | Enforce access control **once**, reuse it (centralized mechanism) | ❌ | ✅ | `improve-codebase-architecture`, `codebase-design` | "Scattered authorization" identified as a deep-module opportunity; deletion test turns scattered checks into a single seam; `implement` says "reuse the existing authz seam" |
| O3 | **Enforce record ownership** (not open CRUD to everyone) | ❌ | ✅ | `implement`, `tdd` | `implement`'s owner-scoping rule (records filtered to the caller's own resource); `tdd`'s "different owner → rejected" test |
| O4 | Domain model should enforce business-limit rules | 🟡 | ✅ | `domain-modeling` | "Model ownership and roles" section: writes ownership + boundary rules to `CONTEXT.md` as invariants/ADRs |
| O5 | Directory listing off; no metadata/backup files in web-root | ❌ | ✅ | `code-review` | Access Control sub-agent brief adds "flag any newly served static/metadata file" item |
| O6 | **Log + alert** on access control failures | ❌ | ✅ | `implement`, `security-review` | `implement`'s "log failed authorization attempts" rule; deep alerting layer stays with the built-in `security-review` |
| O7 | **Rate-limit** API/controller access | ❌ | ✅ | `code-review` | Access Control brief says "flag removed/missing rate-limit" |
| O8 | Invalidate session id on logout; short-lived stateless JWT | 🟡 | ✅ | `security-review` (built-in) + `code-review` | Covered by the built-in `security-review`; `code-review` catches token-lifetime/session-invalidation changes in the diff |
| O9 | Functional **access control tests** (unit/integration) | ❌ | ✅ | `tdd` | Loop rule: a denial test first for every protected seam; `to-spec`'s Testing Decisions also requires this |

---

## 3. Attack scenarios

| # | A01 scenario template | Before | After | Revised skill | What the revision does (what earns the checkmark) |
|---|---|:--:|:--:|---|---|
| S1 | **Parameter tampering (IDOR):** attacker changes the id parameter to access someone else's data | ❌ | ✅ | `tdd`, `code-review` | `tdd` mandates the "cannot access with another owner's id" red test; `code-review` checks "does ownership come from a token" |
| S2 | **Forced browsing:** direct URL access to a privileged endpoint/page | ❌ | ✅ | `code-review` | Access Control axis asks "is there an authz gate + is it fail-closed" on every new/changed endpoint |
| S3 | **Privilege escalation:** an unauthenticated user calls a write/delete endpoint | ❌ | ✅ | `implement`, `tdd` | `implement`'s "gate-before-action + fail-closed" rule; `tdd`'s "unauthenticated → 401/403" denial test |
| S4 | **New endpoint added, authz forgotten** (regression hole) | ❌ | ✅ | `code-review`, `to-spec` | `code-review`'s diff gate: the Access Control sub-agent auto-runs whenever a diff touches an endpoint; `to-spec` records authz for every endpoint at spec time (no room to forget) |
| S5 | **Analytics/report leak:** a bulk-data endpoint returns another user's records | ❌ | ✅ | `tdd` | Loop rule: a "does not return another owner's records" denial test is written first for bulk/report endpoints |

---

## 4. Summary: coverage table (before → after)

| Heading | Before ✅ | Before ❌/🟡 | After ✅ | After ❌ |
|---|:--:|---|:--:|:--:|
| Vulnerability types (8) | 0 | Z1–Z8 (Z6 weak) | 8/8 | — |
| Preventions (9) | 0 | O1–O9 (O4,O8 weak) | 9/9 | — |
| Attack scenarios (5) | 0 | S1–S5 | 5/5 | — |

**Before:** the editable local skills **do not systematically cover** A01; two indirect points
(Z6, O8) only matched the built-in `security-review` — periodic/deep, no gate on **every diff**.
**After:** the P0+P1+P2 revisions give **full coverage** across all three headings; the A01 lens
in `code-review` becomes an inline gate on every PR, and `tdd`'s negative tests give regression protection on every endpoint.

---

## 5. Skill revisions (global, project-independent)

Each revision is added into `~/.claude/skills/<skill>/SKILL.md` as a behavior rule, **without**
any project-specific name/example.

| Priority | Skill | Behavior to add (global) | Items closed |
|---|---|---|---|
| **P0** | `code-review` | **A01 access-control lens:** on every endpoint/handler diff, ask — (a) is there an authz gate? (b) is it scoped to the caller's *own* resource? (c) does the ownership id come **from the token** (not the client)? (d) is it fail-closed? (e) is CORS/rate-limit/directory-listing sane? | Z1,Z2,Z4,Z7,Z8, O5,O7, S2,S4 |
| **P0** | `tdd` | Mandatory **negative authz test** in the red step: for every access-protected endpoint, write "different owner / unauthenticated → rejected" test *first* | Z3, O9, S1, S3, S5 |
| **P1** | `to-spec` | Mandatory **"Access Control" section** on every spec: who can do what, who sees/changes what, ownership boundary, deny-by-default assumption | Z4, Z8, S4 |
| **P1** | `implement` | **Authz-by-default + ownership scoping** when generating endpoints; never trust the client id; log failed authorization | Z2, O3, O6, S3 |
| **P2** | `improve-codebase-architecture` / `codebase-design` | Consolidate access control into a **single enforcement point** (authz seam, deny-by-default); address per-controller duplication | O1, O2, Z1 |
| **P2** | `domain-modeling` | Model **ownership relationships** and roles (who owns what; privilege boundaries) | O4, Z5 |

> `security-review` is not a local skill (it's a built-in command) — it can't be edited. It's assumed to
> cover A01 as a deep/periodic layer; the `code-review` lens complements it with a **fast inline check on every PR**.

---

## 6. Two scopes: diff gate vs. project-wide scan (IMPLEMENTED)

A01 is caught by two different triggers; both were written:

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (third axis: **Access Control**) | **diff** (new hole) | every PR | project-local `.claude/skills/code-review` | ✅ added |
| `audit-access-control` (**NEW**) | **whole project** (existing holes) | pre-release / periodic | **global** `~/.claude/skills/audit-access-control` | ✅ created |
| `security-review` | whole project (general security) | deep scan | built-in | (not edited) |

- **`code-review` / Access Control axis:** runs when the diff touches an endpoint (route/handler/controller/resolver)
  or auth/CORS/rate-limit config; checks the authz gate + ownership scope + token-sourced id
  + fail-closed + privilege-escalation path. It does **not see** out-of-diff holes (a deliberate limit).
- **`audit-access-control`:** inventories every entry point, tabulates the authz state for each
  (endpoint→authz inventory) + project-wide posture (deny-by-default, centralization, CORS, rate-limit,
  logging). Doesn't look at a diff.

**Global note:** `audit-access-control` deliberately lives under `~/.claude/skills/` — since it's a
generic security behavior, it works in every project. The A01 lens in `code-review` currently lives
in this project's copy only; if you want it in other projects too, the same three-axis copy can be
moved to global.

---

## 7. Per-skill analysis: current (before) state of each revised skill

Not just `code-review` — for every skill in the plan, **what state is the current SKILL.md content in
w.r.t. A01**, what the revision will add, and whether it's been applied — in the same before→after format.

### 7.1 `code-review` — ✅ IMPLEMENTED
- **Before:** Two axes (Standards = Fowler-smell baseline; Spec = issue matching). Nowhere in the
  frontmatter/flow/aggregate does authz/access-control/A01 appear. → A01 coverage **❌**.
- **After:** A third axis, **Access Control**, added — a parallel sub-agent runs whenever the diff
  touches an endpoint or auth/CORS/rate-limit config (authz gate + ownership + token-sourced id + fail-closed +
  privilege-escalation). → **✅** (diff gate).

### 7.2 `audit-access-control` — ✅ IMPLEMENTED (new, global)
- **Before:** **None.** No editable skill did a project-wide access-control scan. → **❌**.
- **After:** A new global skill; inventories every entry point, produces an endpoint→authz inventory +
  project-wide posture (deny-by-default, centralization, CORS, rate-limit, logging). → **✅** (project-wide scan).

### 7.3 `tdd` — ✅ IMPLEMENTED
- **Before:** Red→green loop, tests at the seam, anti-patterns (implementation-coupled, tautological,
  horizontal slicing). Says "test behavior through the public interface" but **negative authz testing**
  (unauthorized → rejected) is **not** a concept it names; its "good test" examples are all functional. → A01 test discipline **❌**.
- **After:** An item added to the "rules of the loop" — for every access-protected seam, write **first**
  a "different owner / unauthenticated → 401/403" red test; ownership is proven from the token. The
  **"Authz-blind happy path"** anti-pattern added (if deleting the auth gate breaks no test, there's a hole). → **✅**.

### 7.4 `to-spec` — ✅ IMPLEMENTED
- **Before:** Template: Problem / Solution / User Stories (`As an <actor>...`) / Implementation Decisions /
  Testing / Out of Scope. The actor concept exists but there's no **"who can't do what / ownership
  boundary / deny-by-default"** heading; authz isn't captured at spec time. → **🟡** (actor seed present, no authz section).
- **After:** A mandatory **"Access Control"** section added to the template (allowed/forbidden actions
  per actor, ownership boundary, deny-by-default, privilege levels) + Testing Decisions now requires a
  **denial test** for every access-protected action. → **✅**.

### 7.5 `implement` — ✅ IMPLEMENTED
- **Before:** Minimal: implement the spec, use `/tdd`, typecheck, `/code-review`, commit. No guidance
  toward **authz-by-default / ownership scoping / logging failed authorization** when generating an endpoint. → **❌**.
- **After:** A **"Authorization by default (OWASP A01)"** section added — gate-before-action,
  owner-scoping (id from the token, not the client), fail-closed, log denials, reuse the existing
  authz seam. → **✅**.

### 7.6 `domain-modeling` — ✅ IMPLEMENTED
- **Before:** Glossary sharpening, term conflicts, scenario stress-testing, ADRs. Models relationships
  but **ownership relationships / roles / privilege boundaries** aren't a dedicated lens. → **🟡** (models
  relationships, doesn't name authz).
- **After:** A **"Model ownership and roles (OWASP A01)"** section added — name the ownership relationship
  as an invariant, roles/privilege levels, "can A touch B's resource?" boundary rule; write to `CONTEXT.md`
  as an invariant + ADR if needed. → **✅**.

### 7.7 `improve-codebase-architecture` (+ the `codebase-design` glossary) — ✅ IMPLEMENTED
- **Before:** Deep-module glossary (module/interface/depth/seam/adapter/leverage/locality), deletion test.
  The seam concept **could carry** authz but the idea "access control should consolidate into a single
  enforcement point (authz seam)" **never appears**. → **🟡** (glossary fits, no authz application).
- **After:** A **"scattered authorization"** deepening opportunity added to the Explore step — scattered
  per-handler authz → a single deep **authz seam** (deny-by-default); the deletion test applies.
  (`codebase-design`'s glossary is unchanged, used as the carrier.) → **✅**.

### Summary
| Skill | Before | After | Status |
|---|:--:|:--:|---|
| `code-review` | ❌ | ✅ | ✅ implemented |
| `audit-access-control` | ❌ (didn't exist) | ✅ | ✅ implemented (global) |
| `tdd` | ❌ | ✅ | ✅ implemented |
| `to-spec` | 🟡 | ✅ | ✅ implemented |
| `implement` | ❌ | ✅ | ✅ implemented |
| `domain-modeling` | 🟡 | ✅ | ✅ implemented |
| `improve-codebase-architecture` / `codebase-design` | 🟡 | ✅ | ✅ implemented |

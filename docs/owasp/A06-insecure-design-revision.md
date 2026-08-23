# OWASP A06:2025 — Insecure Design — Skill Revision Document (project-independent)

Purpose: Itemize A06 under three headings (components — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A06 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A06:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

## 0. Methodology note: A06 breaks the pattern

A06 is **fundamentally different** from the previous categories. Definition: "missing or ineffective control design" — insecure design cannot be fixed by perfect implementation, because the necessary controls were never designed in the first place. Root cause: lack of business risk profiling.

Consequence: **there is no grep-able code anti-pattern.** The issue isn't what's "present" in the code, it's what's "missing" from the design. Therefore A06:

- **gets no sub-section added to `code-review`.** Insecure design is about what's "missing"; code-review examines what's present in a diff, and cannot see what's missing from the design.
- **gets no section added to `audit-security`.** A whole-project sweep can scan for something concrete like MD5, but it cannot scan for "insecure design"; the little it could catch (rate-limit, tenant=ownership) already lives in the A01 section.

This is a deliberate scoping decision — not every category takes the same 3-4 skills. A06 goes entirely to the **design-time skills** (`domain-modeling`, `to-spec`), with `tdd` getting a light touch.

## 1. Out-of-scope and conditional items

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| Component 1 (budget/resource) | The budget/resource-planning part of Requirements & Resource Management | Org/management process |
| Component 3 (entire) | Secure Development Lifecycle: paved-road, secure component library, incident post-mortem, OWASP SAMM | Org-level program/asset, not code behavior |
| P1 | Set up an SDLC with AppSec | Org process setup |
| P2 | Secure design pattern library | Org asset |
| P7 (network part) | System/network layer segregation | Infra/deployment architecture |

Conditional/partial:

| # | Item | Condition / boundary |
|---|---|---|
| P5 | Plausibility check at every layer | As a design principle in `domain-modeling`/`to-spec`; overlaps with A05 validation |
| P7 (tenant part) | Tenant segregation | As a domain boundary in `domain-modeling` (parallel to A01 ownership); network part out of scope |
| S3 | Anti-bot design against scalper bots | Anti-abuse design as an abuse case in `to-spec`; the rate-limit part already lives in A01 |

## 2. Components & preventions (in scope: P3, P4, P6 + P5/P7 partial)

| # | A06 prevention | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| P3 | Threat modeling for auth/access-control/business-logic/critical flows | ❌ | ✅ | `to-spec`, `domain-modeling` | to-spec: spec-time "Threat Model / Abuse Cases" section. domain-modeling: invention of misuse/abuse scenarios |
| P4 | Integrate security language into user stories | 🟡 (to-spec already has Access Control + Sensitive Data) | ✅ | `to-spec` | Security acceptance criteria for every abuse case; ties existing security sections to the threat model |
| P6 | Unit/integration tests against the threat model; use-case/misuse-case | 🟡 (tdd already has denial + injection tests) | ✅ | `tdd` (light) | Names the existing denial+injection rules as misuse-case tests; derives tests from the spec's Abuse Cases section |
| P5 (partial) | Plausibility check at every layer | ❌ | 🟡 | `domain-modeling` | "Is this state plausible" invariant check in business-logic flows |
| P7 (partial) | Tenant segregation | 🟡 (A01 ownership) | ✅ | `domain-modeling` | Model tenant as an ownership boundary (cross-tenant access = invariant violation) |

## 3. Attack scenarios (in scope: 2/3, partial: 1/3)

| # | A06 scenario template | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| S1 | "Security question"-based credential recovery (banned by NIST/ASVS/OWASP) | ❌ | ✅ | `to-spec` | Warning as a known-banned design pattern in the Abuse Cases section (**cross-ref with A07** — credential recovery) |
| S2 | Cinema group-booking business-logic flaw (deposit-threshold bypass → mass booking) | ❌ | ✅ | `domain-modeling` | Model business-logic limits as invariants (threshold/limit/quota as a domain rule) |
| S3 (partial) | Missing anti-bot design against scalper bots | ❌ | 🟡 | `to-spec` | Consider "automated/mass abuse" as an abuse case; rate-limit lives in A01 |

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Preventions (7) | 3/7 in scope (P3,P4,P6) + 2/7 partial (P5,P7) | 0 (P4,P6,P7 indirect) | 3/3 (+ 2 partial) |
| Attack scenarios (3) | 2/3 in scope + 1/3 partial | 0 | 2/2 (+ 1 partial) |

Before: none of A06's items were directly covered in the local skill set; to-spec's security sections (A01/A04) and tdd's misuse tests provided an indirect foundation. After: the in-scope items are tied to the design-time skills. A06 is deliberately the most **design-heavy** category — the code skills come away empty.

## 5. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `to-spec` | New **"Threat Model / Abuse Cases"** section: spec-time threat modeling for auth/access-control/business-logic/critical flows; for every important flow, a list of "who could abuse this, and how" abuse cases + a security acceptance criterion for each; known-banned patterns (security-question-based credential recovery — remove). Frames the existing Access Control + Sensitive Data sections as outputs of this threat model | P3, P4, S1, S3(partial) |
| P0 | `domain-modeling` | Extends the existing "Discuss concrete scenarios" step to **inventing misuse/abuse cases** (not just edge cases, but abuse scenarios); models **business-logic limits** (threshold/limit/quota — S2-type) as invariants; adds **tenant segregation** as an ownership boundary (cross-tenant access = invariant violation); a plausibility invariant in business flows | P3, P5(partial), P7(partial), S2 |
| P1 | `tdd` (light) | Names the existing denial (A01) + injection (A05) rules as effectively **misuse-case tests**; suggests deriving additional misuse-case tests from the spec's "Abuse Cases" section | P6 |

**Untouched skills and why:**
- `code-review` — A06 is not grep-able/diff-visible (see §0)
- `audit-security` — a whole-project sweep cannot scan for insecure design (see §0)
- `implement` — already carries P4/P5 indirectly via the existing Access Control / Sensitive Data / Injection sections; a separate A06 rule is unnecessary
- `improve-codebase-architecture` — "secure pattern reuse" is at the org-asset level, not a deep-seam opportunity

## 6. Coverage pattern: why A06 gets no diff/audit scope

A01-A05 had two scopes (code-review diff gate + audit-security project-wide scan). A06 has **neither** — because insecure design is not a code pattern, it's a design-time gap. A06's "gate" is spec-time (`to-spec` threat modeling) and model-time (`domain-modeling` misuse-case + invariant). The catch point is **before** the PR or the release scan, at the design stage.

## 7. Cross-references

- **A06 ↔ A01:** business-logic/access-control limits; tenant segregation parallels ownership (domain-modeling carries both).
- **A06 ↔ A05:** the design dimension of plausibility checking / positive validation (left here in the A05 analysis — now addressed in domain-modeling/to-spec).
- **A06 ↔ A07:** S1 credential recovery (security question) overlaps with A07 authentication — to be confirmed in the A07 analysis.
- **A06 ↔ A09/A10:** the PDF's own table — misuse-case → error/log design. The A09 (logging) and A10 (exceptional conditions) analyses will address how abuse cases feed failure/log design.

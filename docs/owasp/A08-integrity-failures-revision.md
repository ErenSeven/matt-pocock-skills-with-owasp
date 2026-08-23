# OWASP A08:2025 — Software or Data Integrity Failures — Skill Revision Document (project-independent)

Purpose: Itemize A08 under three headings (definition sub-themes — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A08 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A08:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: A08 is a **narrow** category for this skill set — its own core is **insecure deserialization** (a classic code anti-pattern) + **SRI** (CDN integrity) + **in-app update/download signature validation**. The rest is handed off to A02 (CI/CD) and A03 (package source).

Cross-references (PDF): A08 ↔ A03 (package source — **owned by A03**), A08 ↔ A02 (CI/CD — **owned by A02**).

## 0. Cross-ref and out-of-scope items

Owned by another category (A08 does not re-own them, cross-ref only):

| # | Item | Owner | Note |
|---|---|---|---|
| T1 (library) / O2 / S3 | Untrusted library/package source; only trusted repos | **A03** (Supply Chain) | Already covered by `audit-security`'s A03 section + `implement`'s Dependency hygiene |
| T2 / O4 | Unsigned CI/CD artifact; CI/CD segregation/config/access-control | **A02** (Security Misconfiguration) + infra | `audit-security` A02 + CI/CD infra |

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| O3 | Review process for code/config changes | Org process (ironic: the `code-review` skill is the tool for this, but it's not an A08-specific code rule) |
| S2 | Unsigned firmware update (router/IoT) | Firmware/embedded/infra — outside the scope of a general web/app skill set |
| S1 (DNS part) | DNS mapping to a support provider | DNS/infra configuration |

Conditional/partial:

| # | Item | Condition / boundary |
|---|---|---|
| T3 | Auto-update without verification | Code-visible if the app has update logic (covered); otherwise absent |
| S1 (cookie part) | Auth cookies leaking to a 3rd-party domain | Cookie-scope design — indirectly covered by `implement`'s Authentication cookie rule; noted in A08 |

## 1. Definition sub-themes & preventions (in-scope core)

| # | A08 item | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| T4 / O5 / S4 | Insecure deserialization (untrusted serialized data opened without trust/integrity checks → RCE) | ❌ | ✅ | `audit-security` (A08), `code-review`, `implement` | Unsafe-deserializer sink scan (pickle/ObjectInputStream/unserialize/yaml.load/Marshal.load); open untrusted data with a safe format/allowlist; integrity check for client serialized state |
| T1 (CDN) / O1 (SRI) | CDN/external script included without SRI | ❌ | ✅ | `audit-security` (A08), `code-review`, `implement` | Detection of external script/link missing `integrity` (SRI) attribute + rule to add it |
| T3 / O1 (in-app) | In-app auto-update / download-and-execute lacks signature/integrity validation | ❌ | 🟡 | `audit-security` (A08), `code-review`, `implement` | Signature/checksum validation rule for downloaded/updated code-data (if update logic is present) |

## 2. Attack scenarios (in scope: 1/4, cross-ref: 1, partial: 1, out: 1)

| # | A08 scenario template | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| S4 | A Java object is serialized to the client and taken back ("rO0" base64) → RCE via deserialization | ❌ | ✅ | `audit-security` (A08), `code-review`, `implement` | The concrete manifestation of T4/O5 |
| S3 | Unsigned package from an untrusted site → malicious code | — | cross-ref A03 | — | See §0 |
| S1 (cookie) | Auth cookies leaking to a 3rd-party provider | ❌ | 🟡 | `implement` | Cookie-scope note (indirectly via the Authentication section) |
| S2 | Unsigned firmware update | — | out of scope | — | See §0 |

## 3. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Definition/prevention core | 2/5 in scope (deserialization, SRI) + 1 partial (in-app update) + 2 cross-ref | 0 | 2/2 (+ 1 partial) |
| Attack scenarios (4) | 1/4 in scope + 1 cross-ref + 1 partial + 1 out | 0 | 1/1 |

Before: none of A08's items were covered in the local skill set; package-source (A03) provided indirect grounding. After: A08's own core (insecure deserialization + SRI + in-app integrity) is closed; the remaining supply-chain/CI-CD items are already covered in A03/A02.

## 4. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A08 section**) | Insecure-deserialization sink scan (pickle.load, ObjectInputStream/readObject, PHP unserialize, unsafe yaml.load, Ruby Marshal.load, .NET BinaryFormatter); external `<script>`/`<link>` without SRI; missing signature/checksum validation on in-app download/auto-update. Cross-refs package-source→A03, CI/CD→A02 | T4, O5, S4, T1(CDN), O1(SRI), T3 |
| P0 | `code-review` | Security axis → seventh sub-section **Integrity (OWASP A08)** — gate: fires when the diff adds deserialization of untrusted data, an external/CDN script include, or download/update logic. Checks: (a) is untrusted/client-sourced data opened with an unsafe deserializer? (b) does the external script carry `integrity` (SRI)? (c) is the signature/integrity of downloaded/updated content validated? (d) is there an integrity check on client-supplied serialized state? | T4, O5, S4, T1(CDN), O1, T3 |
| P1 | `implement` | New **"Integrity (OWASP A08)"** section: don't open untrusted data with an unsafe deserializer — use a safe format (JSON) / allowlisted deserializer; add SRI to external scripts; validate the signature/checksum of everything downloaded/auto-updated; don't trust client-supplied serialized state without a signature/integrity check | T4, O5, S4, T1(CDN), O1, T3, S1(cookie) |

**Untouched skills:**
- `tdd` — insecure-deserialization payload testing can already be derived under the umbrella of the A05 (injection) / A06 (misuse-case) test rules; a separate rule is unnecessary (user decision: leave untouched)
- `to-spec`, `domain-modeling` — A08 has no spec-actor/domain-ownership dimension
- `improve-codebase-architecture` — no A08-specific deep-seam opportunity

## 5. Two scopes: diff gate vs. project-wide scan (same pattern as A01-A05, A07)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Integrity sub-section) | diff (new/open) | every PR | `skills/code-review` | implemented |
| `audit-security` (A08 section) | whole project (existing/open) | pre-release / periodic | `skills/audit-security` | implemented |

## 6. Notes

- **A03 boundary:** the trustworthiness of the package/dependency source belongs to A03; A08 forward-refs it. A08's contribution is **verifying integrity** (signature/checksum/SRI) even when the source is trusted, and safely handling untrusted **data** (serialized payloads).
- **A02 boundary:** CI/CD security belongs to A02/infra; A08 focuses only on in-app download/update/deserialization code.

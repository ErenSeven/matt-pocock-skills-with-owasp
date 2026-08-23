# OWASP A02:2025 — Security Misconfiguration — Skill Revision Document (project-independent)

Purpose: itemize A02 under three headings (symptoms/vulnerability types — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: this document is not tied to any specific project. "Covered" = a skill's current instructions address this A02 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md` (from which both the `~/.claude/skills` symlink and the Copilot output via `scripts/generate-copilot.ps1` are produced).

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — A02:2025 section, successfully extracted with `pdftotext`.

Status legend: ✅ covered · 🟡 indirect/weak · ❌ not covered · 🚫 out of scope (not this skill set's domain)

Columns: Before = current skill state · After = state once the revision is applied

## 0. Items excluded from scope (will not be implemented)

This skill set is project-independent and code-focused (writing/reviewing code, specs, domain modeling). The following A02 items are pure infrastructure/cloud-console/network operations with **no code-side counterpart**, so they're excluded from scope:

| # | Item | Why out of scope |
|---|---|---|
| B5 | Current security features disabled | Too generic/vague — its concrete actions already spread into B7/B8, doesn't produce a separate revision as its own item |
| O3 (patch part) | Patch management | A dependency/library freshness topic — belongs to **A03 Software Supply Chain Failures**, handed off there (cross-reference) |
| O3 (cloud permission part) | Cloud storage permission auditing | Pure cloud-console IAM/S3-bucket-policy work, invisible in the repo unless there's IaC |
| O4 | Segmented architecture (network/container segmentation) | Network/infra architecture — not the "code module depth" that `improve-codebase-architecture` covers |
| S4 | Cloud storage permissions open to the internet by default | S4 is the concretized form of O3's cloud-permission part — out of scope for the same reason |

Conditional (covered only if the repo has IaC — Terraform/CloudFormation/Pulumi etc. — not a mandatory item):

| # | Item | Condition |
|---|---|---|
| B1 | Missing hardening / misconfigured cloud permissions | If IaC files are in the repo, `audit-security` also scans them; otherwise skipped |
| O1 | Repeatable automated hardening; same config across dev/QA/prod with different credentials | Covered if config-as-code is in the repo; the deploy pipeline itself is outside this skill set |

## 1. Symptoms / common vulnerability types (in scope: 6/8)

| # | A02 symptom | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| B2 | Unnecessary ports/services/pages/accounts/test frameworks active | ❌ | ✅ | `audit-security` (A02 section), `code-review` | Audit: project-wide sample/demo/debug route scan. code-review: flags a newly added debug/test/sample endpoint in the diff |
| B3 | Default account/password unchanged | ❌ | ✅ | `audit-security` (A02 section), `code-review` | Audit: hardcoded default-credential grep. code-review: flags a newly entered default credential/static secret in the diff |
| B4 | No centralized error handling; stack trace leaks to the user | ❌ | ✅ | `implement`, `code-review` | implement: global/central exception-handler rule — stack traces never return to the client. code-review: checks whether a new error handler leaks stack traces |
| B6 | Insecure configuration kept for backward compatibility | ❌ | ✅ | `code-review`, `audit-security` | code-review: flags the pattern "old insecure mode/flag left on for backward compatibility" in the diff. Audit: project-wide legacy-insecure-config scan |
| B7 | Server/framework/library/DB not set to secure values | ❌ | ✅ | `audit-security` (A02 section), `implement` | Audit: scans config files against a secure-values checklist. implement: apply secure defaults when setting up a new server/framework/DB |
| B8 | Missing security headers/directives | ❌ | ✅ | `implement`, `code-review`, `audit-security` (A02 section) | implement: a default security-header set on new server/route setup. code-review: flags a diff that weakens/removes header middleware. Audit: project-wide header inventory |

Out of scope: B1 (conditional), B5 (removed — see §0).

## 2. Preventions (in scope: 3/7, patch part handed off to A03)

| # | A02 prevention | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| O2 | Minimal platform; remove unnecessary features/sample apps | ❌ | ✅ | `audit-security` (A02 section) | Project-wide sample/demo-app and unused-feature scan, feeds the same findings as B2 |
| O5 | Send security headers | ❌ | ✅ | `implement` | "Secure defaults" section: security headers added by default on new server/route setup |
| O6 | Automated verification process; if none, manual verification once a year | ❌ | ✅ | `audit-security` (A02 section) | This prevention's counterpart is literally the new skill's reason for existing — a "run periodically/pre-release" note in the skill description |
| O7 | Identity federation / short-lived credentials / role-based access instead of static keys/secrets | ❌ | ✅ | `implement`, `code-review`, `audit-security` (A02 section) | implement: point to env/secret-store instead of hardcoded secret/static key. code-review: flags a newly hardcoded secret in the diff. Audit: project-wide static-secret scan. **Note:** key/rotation mechanics overlap with A04 (Cryptographic Failures) — cross-reference |

Out of scope: O1 (conditional), O3 (removed — patch → A03, cloud permission → out of scope), O4 (removed).

## 3. Attack scenarios (in scope: 3/4)

| # | A02 scenario template | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| S1 | Un-removed sample app + default admin password → server takeover | ❌ | ✅ | `audit-security` (A02 section), `code-review` | The concretized form of B2+B3; closed by the same revisions |
| S2 | Directory listing open → compiled files downloaded and decompiled, holes found | 🟡 (previously partially in `audit-access-control`'s A01 posture section) | ✅ | `audit-security` (A02 section) | The directory-listing/metadata-file check is removed from the A01 posture section and moved to A02 as its true owner (single file, single place) |
| S3 | Detailed stack trace returned to the user → info/version disclosure | ❌ | ✅ | `implement`, `code-review` | The concretized form of B4; closed by the same revisions |

Out of scope: S4 (removed — see §0).

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Symptoms (8) | 6/8 (B1 conditional, B5 removed) | 0 | 6/6 |
| Preventions (7) | 3/7 (O1 conditional, O3+O4 removed) | 0 | 3/3 |
| Attack scenarios (4) | 3/4 (S4 removed) | 0 (S2 weak/indirect) | 3/3 |

Before: none of A02's vulnerability types were directly covered by the local skill set (only S2 was indirectly present, in `audit-security`'s A01 posture section). After: the P0+P1 revisions close all 12/12 in-scope items.

## 5. Skill revisions (project-independent)

Each revision is added into `skills/<skill>/SKILL.md` as a behavior rule, without any project-specific name/example.

| Priority | Skill | Behavior added (implemented — d651bea) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A02 section**, alongside the existing A01 section — project-wide sweep) | Sample/demo/debug endpoint inventory, default-credential/static-secret scan, legacy-insecure-config scan, server/framework/DB secure-values checklist, security-header inventory, directory-listing check. Runs periodically/pre-release, doesn't look at a diff | B2, B3, B6, B7, B8, O2, O6, O7, S1, S2 |
| P0 | `code-review` | Rather than opening a new axis, a second sub-section inside the existing **Security** axis: **Security Config (OWASP A02)** — its own gate (diff touches a new/changed entry point, error handler, middleware, or server config file) and its own checklist (was a debug/sample/test endpoint added? was a default/static credential entered? does a stack trace/detailed error return to the client? was an old insecure mode left on "for backward compatibility"? was security-header middleware weakened/removed?), added alongside the Access Control (A01) sub-section — the number of agents stays fixed at 3 | B2, B3, B4, B6, B8, O7, S1, S3 |
| P1 | `implement` | "Secure defaults (OWASP A02)" section: central/global exception handler (stack traces never return to the client), a default security-header set on new server/route setup, point to env/secret-store instead of a hardcoded default credential/static secret | B4, B7, B8, O5, O7, S3 |

The directory-listing item now lives in exactly one place in one file: removed from A01's posture checklist and moved to A02, its true owner (the overlap that existed while the skills were separate became unnecessary once merged).

## 6. Two scopes: diff gate vs. project-wide scan (same pattern as A01)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Security Config sub-section) | diff (new hole) | every PR | `skills/code-review` | implemented (d651bea) |
| `audit-security` (A02 section, alongside the existing A01 section) | whole project (existing hole) | pre-release / periodic | `skills/audit-security` | implemented (d651bea) |

`code-review`'s Security axis now carries two sub-sections — **Access Control (A01)** and **Security Config (A02)** — one sub-agent, one report heading, each sub-section with its own gate. The Security Config sub-section runs when the diff touches an entry point, error handler, middleware, or server/config file; it checks for debug/sample endpoints + default credentials + stack-trace leaks + legacy insecure config + weakened headers. It doesn't see out-of-diff holes (a deliberate limit).

`audit-security`: inventories the entire sample/demo/debug surface, default credentials, security headers, and directory-listing/legacy-config state. Doesn't look at a diff.

## 7. Open decisions

1. ~~Adding a fourth parallel axis to `code-review`~~ — **decided (2026-08-17): the axis count will not grow.** A02 was added as a second sub-section inside the existing Security axis (formerly named Access Control); the sub-agent count stays fixed at 3. This will also be the default approach for A03–A10 — a new top-level axis only opens if a category genuinely needs isolation (see `skills/code-review/SKILL.md` → "Why Security stays one axis, not one-per-category").
2. ~~New skill name `audit-security-config`~~ — **decided (2026-08-17): no separate skill will be opened.** `audit-access-control` was renamed to `audit-security` to cover all ten categories; A02 wasn't made a separate skill, it was added as this skill's second section. This will also be the default approach for A03–A10 (see the note at the end of `skills/audit-security/SKILL.md`).
3. **Handoff of O3's patch-management part to A03** — will be revisited there once the A03 (Software Supply Chain Failures) analysis begins; just noted here for now.

# OWASP A07:2025 — Authentication Failures — Skill Revision Document (project-independent)

Purpose: Itemize A07 under three headings (symptoms — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A07 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A07:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: A07 is code-heavy (session management, JWT validation, error messages, brute-force lockout, session fixation are all code-visible) → the same 3-code-skill pattern as A02/A04/A05. But the cross-references left with A04/A06/A02 converge here.

Cross-references (PDF): A07 ↔ A04 (password hash storage — **owned by A04**), A07 ↔ A01 (session/JWT). Additionally: A07 ↔ A06 (S1 credential recovery), A07 ↔ A02 (default credentials).

## 0. Cross-ref and out-of-scope items

Items owned by another category (A07 does not re-own them, only cross-references):

| # | Item | Owner | Note |
|---|---|---|---|
| B6 | Plaintext/weak-hash password storage | **A04** (Cryptographic Failures) | The PDF explicitly says "see A04" — already covered by `audit-security`'s A04 section + `implement`'s Cryptography section |
| O3 | Deploying with default credentials | **A02** (Security Misconfiguration) | Already covered by `audit-security` A02 + `implement` Secure defaults |
| B5 | Weak credential recovery (security question) | **A06** (Insecure Design) — confirmed | `to-spec`'s Threat Model catches this as "banned-by-design"; A07 also flags it on the code side (recovery flow) |

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| O2 | Encourage password manager use | UX/product decision; the only code-facing angle ("don't block paste / support long passwords") doesn't warrant a general behavior rule |

Conditional/partial (feature or policy — not core):

| # | Item | Condition / boundary |
|---|---|---|
| B1 | Credential stuffing / password spray exposure | Defense = rate-limit (code) + breach detection (feature); the rate-limit part is covered |
| B4 / O5 | Leaked credentials / haveibeenpwned breach-list validation | External service integration/feature — `implement`/`audit` note "is this present," they don't set up the integration |
| B7 / O1 | Missing MFA / mandate MFA | MFA implementation is code, "mandate" is policy — `implement` says "prefer/support MFA," org policy is out of scope |

## 1. Symptoms (in scope: 7/11, cross-ref: 1, partial: 3)

| # | A07 symptom | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| B2 | Brute force isn't blocked quickly | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement` | Failed-login lockout/delay (overlaps with A01 rate-limit posture — here it's auth-specific) |
| B3 | Default/weak/known passwords accepted | ❌ | ✅ | `audit-security` (A07), `implement` | Registration-time password policy (weak-password list) |
| B8 | Session ID exposed in URL/hidden field | ❌ | ✅ | `audit-security` (A07), `code-review` | Detection of session-id exposure in URL/hidden field |
| B9 | Same session ID after login (session fixation) | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement`, `tdd` | Detection/rule/test for session-id not being renewed at login → fixation |
| B10 | Session not invalidated on logout/inactivity | ❌ | ✅ | `audit-security` (A07), `implement` | Logout/idle/absolute-timeout invalidation (overlaps with A01 posture) |
| B11 | JWT aud/iss/scope not correctly asserted | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement` | JWT claim + signature + expiry validation |
| B5 | Weak credential recovery (security question) | 🟡 (A06 to-spec) | ✅ | `audit-security` (A07), `code-review` | Detection of security-question pattern in recovery flow (A06 cross-ref) |
| B1 (partial) | Credential stuffing / spray | ❌ | 🟡 | `audit-security` (A07), `implement` | Rate-limit + generic-error part |
| B4 (partial) | Account with leaked credentials | ❌ | 🟡 | `implement` | Breach-list check is noted as "is this present" |
| B7 (partial) | Missing/ineffective MFA | ❌ | 🟡 | `implement` | Support/prefer MFA |
| B6 | Plaintext/weak hash | — | cross-ref A04 | — | See §0 |

## 2. Preventions (in scope: 6/11, cross-ref: 1, partial: 3, out: 1)

| # | A07 prevention | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| O4 | Test against the top 10,000 weak-password list | ❌ | ✅ | `implement`, `audit-security` (A07) | Weak-password rejection |
| O6 | NIST 800-63b password policy; mandatory rotation | ❌ | ✅ | `implement` | NIST-compliant policy + **flags mandatory rotation/complexity enforcement** (S2) |
| O7 | Same error message against enumeration | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement`, `tdd` | Generic auth error; enumeration test |
| O8 | Failed-login limit/delay + log + alert | ❌ | ✅ | `audit-security` (A07), `implement` | Lockout + logging (overlaps with A01 access-failure log + A09) |
| O9 | Server-side session manager: new high-entropy ID, secure cookie, timeout invalidation | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement` | **The heart of A07** — session-management rule/scan |
| O10 | Prefer a proven/tested auth system | ❌ | ✅ | `implement` | "Don't write your own auth" (parallel to the crypto rule) |
| O11 | Validate JWT aud/iss/scope | ❌ | ✅ | `audit-security` (A07), `code-review`, `implement` | JWT validation (same as B11) |
| O1 (partial) | Mandate MFA | ❌ | 🟡 | `implement` | Prefer MFA |
| O5 (partial) | Breach-list validation | ❌ | 🟡 | `implement` | Noted as "is this present" |
| O3 | Deploying with default credentials | — | cross-ref A02 | — | See §0 |
| O2 | Encourage password manager | — | out of scope | — | See §0 |

## 3. Attack scenarios (in scope: 3/3)

| # | A07 scenario template | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| S1 | Credential stuffing/spray → a defenseless app becomes a "password oracle" | ❌ | ✅ | `implement`, `audit-security` (A07) | Rate-limit + generic error + MFA preference |
| S2 | Single factor + mandatory rotation → encourages weak passwords (NIST doesn't recommend rotation, it recommends MFA) | ❌ | ✅ | `implement` | Design fix: flag mandatory rotation/complexity enforcement, prefer MFA |
| S3 | Missing SSO Single Logout → session stays open on a shared computer | ❌ | ✅ | `audit-security` (A07), `implement` | Logout should invalidate all session/SSO tokens |

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Symptoms (11) | 7/11 in scope + 1 cross-ref + 3 partial | 0 | 7/7 (+ 3 partial) |
| Preventions (11) | 6/11 in scope + 1 cross-ref + 2 partial + 1 out | 0 | 6/6 (+ 2 partial) |
| Attack scenarios (3) | 3/3 | 0 | 3/3 |

Before: A07's authentication-specific items (session fixation, JWT assertion, enumeration, brute-force lockout, credential recovery) were not covered in the local skill set; password-hashing (A04) and default-cred (A02) provided indirect grounding. After: 16/16 in-scope items + 5 partial items are closed.

## 5. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A07 section**) | Auth-flow scan: is session-id in the URL, is session renewed at login (fixation), logout/timeout invalidation, JWT validation (aud/iss/exp/signature), enumeration-friendly error messages, brute-force lockout, password policy, security-question recovery. Cross-refs hashing→A04, default-cred→A02 (no re-scan) | B2, B3, B5, B8, B9, B10, B11, O4, O7, O8, O9, O11, S3 |
| P0 | `code-review` | Security axis → sixth sub-section **Authentication (OWASP A07)** — gate: fires when the diff touches login/logout, session management, JWT handling, password policy, or credential recovery. Checks: (a) is session-id renewed at login (fixation)? (b) is session-id in the URL/hidden field? (c) is JWT signature+aud+iss+exp validated? (d) are auth errors generic (enumeration)? (e) is a security-question recovery being added? | B8, B9, B11, B5, O7, O9, O11 |
| P1 | `implement` | New **"Authentication (OWASP A07)"** section: use a vetted auth/identity/session system (don't write your own auth), renew session at login, invalidate on logout/idle/absolute-timeout, Secure/HttpOnly cookies, validate JWT signature+claims+expiry, generic auth error (anti-enumeration), brute-force lockout+log, NIST-compliant password policy (**flag** mandatory rotation/complexity enforcement), prefer/support MFA | B2, B3, B7(partial), B10, B11, O1(partial), O4, O6, O7, O8, O9, O10, O11, S1, S2, S3 |
| P1 | `tdd` (light) | Adds authentication-specific examples to the existing denial (A01) rule: **session fixation** (session-id must change before/after login) and **enumeration** (unknown user vs. wrong password → same error/timing) tests — auth examples of misuse-case testing for auth-protected seams | B9, O7 |

**Untouched skills:**
- `to-spec` — the existing Access Control (privilege levels) + Threat Model (credential-stuffing abuse case, banned-by-design recovery) sections already carry design-time auth (user decision: leave as is)
- `domain-modeling` — authentication is not a domain-ownership dimension
- `improve-codebase-architecture` — no auth-specific deep-seam opportunity (the existing authz seam lives in A01)

## 6. Two scopes: diff gate vs. project-wide scan (same pattern as A01-A05)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Authentication sub-section) | diff (new/open) | every PR | `skills/code-review` | implemented |
| `audit-security` (A07 section) | whole project (existing/open) | pre-release / periodic | `skills/audit-security` | implemented |

## 7. Notes

- **A01 overlap:** session invalidation, rate-limit, and access-failure logging items also appear in A01 posture. The A07 section addresses them with auth-flow-specific mechanics (login session renewal, brute-force lockout, enumeration); the general session/rate-limit items in A01 posture are left untouched (deliberate, narrow overlap — each meaningful in its own context).
- **A04 boundary confirmed:** password hash storage belongs to A04; A07 forward-refs it without re-writing the rule.
- **A09 connection:** failed-login logging/alert (O8) overlaps with A09 (Logging & Alerting) — the sufficiency of this logging will be addressed in the A09 analysis.

# OWASP A09:2025 — Security Logging & Alerting Failures — Skill Revision Document (project-independent)

Purpose: Itemize A09 under three headings (symptoms — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A09 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A09:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: A09 **splits in two** — the code side of log/alert (what's being logged, is PII leaking, log injection, alert emission) is in scope; the alerting pipeline (SIEM/SOC/thresholds/escalation/playbook/incident-response/tooling) is pure ops/infra, out of scope.

Cross-references (PDF): A09 ↔ A01 (access logging), A09 ↔ A10 (error logging). Additionally: A09 ↔ A07 (failed-login logging), A09 ↔ A04 (PII/sensitive data).

## 0. Cross-ref and out-of-scope items

Left to A10 (user decision):

| # | Item | Owner | Note |
|---|---|---|---|
| B10 | Errors/exceptions aren't handled → can't be logged | **A10** (Exceptional Conditions) | Exception handling is A10's core; A09 only cross-refs the side of "handled errors must be logged" |
| O5 | Failing transaction rolls back + fails closed | **A10** | Fail-closed/rollback will be addressed in A10 |

Out of scope (will not be implemented — ops/infra/tooling):

| # | Item | Why out of scope |
|---|---|---|
| B4 | Logs aren't monitored; local, no backup | Log aggregation/backup infra |
| B5 | No alert threshold/escalation | Ops/SOC |
| B6 | DAST/pentest doesn't trigger alerts | Ops/tooling |
| B7 | No real-time attack detection | SIEM/ops |
| B11 | Alert use cases missing/stale | Ops |
| B12 | Too many false positives → SOC burden | Ops |
| B13 | Playbook missing/stale | Ops |
| O7 | SOC monitoring/alerting use cases + playbook | Ops |
| O9 | Behavioral analysis/AI | Ops/optional |
| O10 | NIST 800-61r2 incident response plan | Ops/org |
| O11 | ModSecurity CRS, ELK stack | Tooling/infra |

Conditional/partial:

| # | Item | Condition / boundary |
|---|---|---|
| B3 / O4 | Log integrity; audit trail (append-only) | Append-only design intent lives in `implement`; storage-integrity infra is out of scope |
| O6 | Alert on suspicious behavior | Emitting the alert **signal** in code is in scope; the alerting pipeline is ops (out of scope) |
| O8 | Honeytoken traps | Niche feature/design — noted, not mandatory |

## 1. Symptoms (in scope: 4/13, to A10: 1, partial: 1)

| # | A09 symptom | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| B1 | Login/failed-login/high-value transactions inconsistently logged | ❌ | ✅ | `audit-security` (A09), `code-review`, `implement` | Security-event logging (overlaps with A07/A01 logging items — this is the true owner) |
| B2 | Errors/warnings logged with insufficient/unclear context | ❌ | ✅ | `implement` | Logging with sufficient context (who/what/when/outcome) |
| B8 | Logs visible to users (A01) or PII/PHI being logged | ❌ | ✅ | `audit-security` (A09), `code-review`, `implement` | Ban on PII/secrets in logs + logs must not leak to users (A01/A04 cross-ref) |
| B9 | Not encoding against log injection | ❌ | ✅ | `audit-security` (A09), `code-review`, `implement` | Encode untrusted data going into log messages (CRLF/forged-entry) |
| B3 (partial) | Log integrity not preserved | ❌ | 🟡 | `implement` | Append-only/audit-trail intent; storage infra out of scope |

## 2. Preventions (in scope: 3/11, to A10: 1, partial: 2)

| # | A09 prevention | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| O1 | Log login/access-control/input-validation failures with sufficient context, retain for forensics | ❌ | ✅ | `audit-security` (A09), `implement` | Security-decision logging rule |
| O2 | Log both success and failure of every security control | ❌ | ✅ | `audit-security` (A09), `implement` | Log both success and failure |
| O3 | Log-management-compliant format; encode against injection | ❌ | ✅ | `implement`, `code-review` | Structured logging + encoding (same as B9) |
| O4 (partial) | Audit trail + integrity (append-only) | ❌ | 🟡 | `implement` | Append-only intent |
| O6 (partial) | Alert on suspicious behavior | ❌ | 🟡 | `implement` | Emit an alert signal (pipeline is ops) |

## 3. Attack scenarios

All are named real-world breach cases (evidence/motivation, like A03 — no separate code rule is produced): a children's health plan undetected for 7+ years (3.5M records), an Indian airline with 10+ years of data exposure, a European airline with 400K+ payment records and a €20M GDPR fine. Lesson: without logging/detection, breaches stay hidden for years. This motivates the category; indirectly addressed via the O1/O2 revisions.

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Symptoms (13) | 4/13 in scope + 1 to A10 + 1 partial + 7 ops-out | 0 | 4/4 (+ 1 partial) |
| Preventions (11) | 3/11 in scope + 1 to A10 + 2 partial + 5 ops-out | 0 | 3/3 (+ 2 partial) |
| Scenarios (3) | motivation (indirect) | 0 | — |

Before: A09's logging items were scattered/indirect in the local skill set (A01 access-failure log, A07 failed-login log). After: an A09 section is established as the true owner of the logging cross-cutting concern; the logging items in A01/A07 remain as contextual references.

## 5. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A09 section**) | Are security events (auth success/failure, access-denial, validation failure, high-value transactions) being logged; do logs contain PII/secrets/full card numbers; is untrusted data going into logs encoded (log injection); is log output leaking to users. True owner of the A01/A07 logging items | B1, B8, B9, O1, O2 |
| P0 | `code-review` | Security axis → eighth sub-section **Logging (OWASP A09)** (light diff-gate) — gate: fires when the diff adds a new security-sensitive action (auth/access-decision/high-value transaction) or adds/changes a log statement. Checks: (a) does the new sensitive action have an audit log? (b) does a new log statement leak PII/secrets? (c) is user input going into a log message encoded (log injection)? | B1, B8, B9, O3 |
| P1 | `implement` | New **"Logging & alerting (OWASP A09)"** section: log security events with sufficient context (who/what/outcome); never log secrets/PII/full card numbers; encode untrusted data in log messages (log injection); emit an alert signal on suspicious events; audit records with append-only intent. Note: catching/logging exceptions and fail-closed/rollback are in A10 | B1, B2, B3(partial), B8, B9, O1, O2, O3, O4(partial), O6(partial) |

**Untouched skills:**
- `tdd` — logging is not naturally a misuse-case/negative test, doesn't fit the TDD loop
- `to-spec`, `domain-modeling` — the existing Sensitive Data section already implies "don't store/leak unnecessarily"; logging detail is an implementation concern
- `improve-codebase-architecture` — logging is cross-cutting but there's no A09-specific deep-seam opportunity (a centralized logger is already common practice)

## 6. Two scopes: diff gate vs. project-wide scan

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Logging sub-section) | diff (new/open) | every PR | `skills/code-review` | implemented |
| `audit-security` (A09 section) | whole project (existing/open) | pre-release / periodic | `skills/audit-security` | implemented |

The Logging sub-section in `code-review` is deliberately **light** — most of A09 is what's "missing" (not logged), which isn't fully visible in a diff (similar boundary to A06). What it catches: a new PII-leaking/injectable log statement + a new sensitive action that isn't logged. The project-wide "which events aren't logged" table lives in `audit-security`.

## 7. Notes

- **A01/A07 overlap:** "access-failure logging" (A01 posture) and "failed-login logging" (A07) are each instances of A09. The A09 section is the cross-cutting owner of logging; the contextual references in A01/A07 remain untouched (each meaningful in its own context).
- **A10 boundary:** exception handling + fail-closed + rollback were left to A10. A09 only carries the "handled events/errors **must be logged**" side; "how errors should be handled" belongs to A10.
- **A04 overlap:** the ban on PII/secrets in logs aligns with A04's "sensitive data" logic; it is repeated here specific to the logging context (logs are a frequently overlooked leak channel).

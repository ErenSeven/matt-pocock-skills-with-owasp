# OWASP A05:2025 — Injection — Skill Revision Document (project-independent)

Purpose: Itemize A05 under three headings (symptoms — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: This document is not tied to any specific project. "Covered" = a skill's current instructions address this A05 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — the A05:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: A05, like A04, is **broad in scope** — a classic static-analysis category, almost entirely a code-visible injection sink.

Cross-reference (PDF): A05 ↔ A06 (input validation **design**). This document covers A05's implementation-level part (validate + parametrize + encode); the design-level part of validation (designing validation into the system from the start) is left to the A06 (Insecure Design) analysis.

## 0. Out-of-scope and conditional items

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| T3 | SAST/DAST/IAST in CI/CD | Tooling/infrastructure setup-and-operation, not code behavior |
| — | LLM Prompt Injection | The PDF puts this on a **separate list** (LLM01:2025) — not part of this OWASP Top 10 pass |

Conditional:

| # | Item | Condition / boundary |
|---|---|---|
| T2 | Automated testing/fuzzing | Fuzzing infrastructure is out of scope; but "malicious-input testing" enters `tdd` as a payload-testing rule |
| O2 | Positive server-side input validation | The implementation-level part (validate) is covered; the **design-level** part (design validation into the system from the start) is left to A06 — cross-ref |

## 1. Symptoms (in scope: 4/4)

| # | A05 symptom | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| B1 | User data is not validated/filtered/sanitized | ❌ | ✅ | `audit-security` (A05), `code-review`, `implement` | Detection of unsanitized/unvalidated user-input flows |
| B2 | Dynamic query/non-parametrized call without context-aware escaping | ❌ | ✅ | `audit-security` (A05), `code-review`, `implement` | Detection of dynamic queries without parametrization |
| B3 | Unsanitized data in ORM search parameters | ❌ | ✅ | `audit-security` (A05), `code-review` | Detection of untrusted data in raw/dynamic ORM queries |
| B4 | Malicious data concatenated directly | ❌ | ✅ | `audit-security` (A05), `code-review`, `implement` | Detection of string-concatenated query/command |

**Injection types (detection targets):** SQL, NoSQL, OS command, ORM/HQL, LDAP, EL/OGNL, **+ Browser/XSS** (the "browser" interpreter in the definition → output encoding). All in scope. LLM Prompt Injection is out of scope (see §0).

## 2. Detection (in scope: 1/3, conditional: 1/3)

| # | A05 detection method | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| T1 | Source code review | ❌ | ✅ | `audit-security` (A05), `code-review` | Already the nature of these two skills; injection-sink scanning added |
| T2 (conditional) | Automated testing/fuzzing | ❌ | 🟡 | `tdd` | Injection-payload test for seams where input reaches an interpreter (not fuzzing infrastructure) |

T3 (CI/CD SAST/DAST/IAST) is out of scope — see §0.

## 3. Preventions (in scope: 3/3)

| # | A05 prevention | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| O1 | Separate data from commands: safe API / parametrize / ORM | ❌ | ✅ | `implement`, `code-review` | Parametrized query / safe API rule; string-concat is flagged |
| O2 | Positive server-side input validation | ❌ | ✅ (implementation-level) | `implement` | Server-side allowlist validation rule; **design-level → A06** — cross-ref |
| O3 | Interpreter-specific escaping for remaining dynamic queries (table/column names cannot be escaped!) | ❌ | ✅ | `implement`, `code-review` | Context-aware escaping rule + warning for unescapable positions (table/column names) |

## 4. Attack scenarios (in scope: 3/3)

| # | A05 scenario template | Before | After | Revised skill | What the revision does |
|---|---|---|---|---|---|
| S1 | Classic SQL injection via `' OR '1'='1` → all records returned | ❌ | ✅ | `tdd`, `audit-security` (A05), `code-review` | tdd: SQL-payload test; audit/review: string-concat SQL sink |
| S2 | Filter bypass via the same logic in HQL (ORM injection) | ❌ | ✅ | `audit-security` (A05), `code-review` | Detection of raw HQL/ORM queries |
| S3 | OS command injection via `; cat /etc/passwd` | ❌ | ✅ | `tdd`, `audit-security` (A05), `code-review`, `implement` | tdd: command-payload test; audit/review/implement: user-input-driven shell exec |

## 5. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Symptoms (4) | 4/4 | 0 | 4/4 |
| Detection (3) | 1/3 in scope + 1/3 conditional | 0 | 1/1 (+ 1 conditional) |
| Preventions (3) | 3/3 | 0 | 3/3 |
| Attack scenarios (3) | 3/3 | 0 | 3/3 |

Before: none of A05's items were covered in the local skill set. After: 11/11 in-scope items are closed. Along with A04, this is among the broadest-scope categories — injection is almost entirely code-visible.

## 6. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A05 section**) | Injection-sink scanning: string-concat SQL/query, dynamic query without parametrization, user-input-driven OS command (`exec`/`system`/`eval`/backtick), raw ORM/HQL query, template injection, missing output encoding (XSS) — sink inventory by type (SQL/NoSQL/OS/ORM/LDAP/EL/XSS) | B1, B2, B3, B4, T1, S1, S2, S3 |
| P0 | `code-review` | Security axis → fifth sub-section **Injection (OWASP A05)** — gate: fires when the diff touches a DB query, OS/shell command execution, template rendering, or code that passes user input to an interpreter. Checks: (a) is the query parametrized, or string-concatenated? (b) is the OS command built with user input? (c) does a raw ORM/HQL query contain untrusted data? (d) is output going to the browser context-aware encoded (XSS)? (e) is there server-side validation? | B1, B2, B3, B4, T1, O1, O3, S1, S2, S3 |
| P1 | `implement` | New **"Injection prevention (OWASP A05)"** section: separate data from commands (parametrized query / safe API / ORM — never concat), positive server-side allowlist validation, context-aware output encoding, warning for unescapable positions (table/column names) | B1, B2, B4, O1, O2, O3, S3 |
| P1 | `tdd` | Item added to rules-of-the-loop (parallel to the A01 authz test): for seams where input reaches an interpreter (DB/shell/template/LDAP), write an **injection-payload test** — malicious input (`' OR '1'='1`, `; cat /etc/passwd`) must be treated as data, never executed as a command. Added to the anti-pattern list: "injection-blind happy path" (an interpreter seam tested only with clean input) | T2, S1, S3 |

`to-spec`, `domain-modeling`, `improve-codebase-architecture` are untouched in this category — injection is an implementation concern (no spec-actor or domain-ownership dimension); the answer to "which input is untrusted" is universal ("all external input"), unchanged per spec. The design dimension of validation is left to A06.

## 7. Two scopes: diff gate vs. project-wide scan (same pattern as A01-A04)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Injection sub-section) | diff (new/open) | every PR | `skills/code-review` | implemented |
| `audit-security` (A05 section) | whole project (existing/open) | pre-release / periodic | `skills/audit-security` | implemented |

The Injection sub-section in `code-review` runs when the diff touches a query/command/template/interpreter-input. It does not see out-of-diff holes.

The A05 section of `audit-security` scans the entire codebase for injection sinks. It does not look at a diff.

## 8. Notes

- **A06 boundary:** O2 (input validation) is covered here at the implementation level; at the design level (threat modeling, designing validation into the system from the start) it will be addressed in A06. This boundary will be confirmed when the A06 analysis begins.
- **tdd pattern:** This is the second security-test rule added to tdd (the first was the A01 denial test). Both rules follow the same structure: for a protected/risky seam, write a negative test first (unauthorized→rejected / bad-input→treated-as-data).

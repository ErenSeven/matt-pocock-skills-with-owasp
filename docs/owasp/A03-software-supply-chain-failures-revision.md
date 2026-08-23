# OWASP A03:2025 — Software Supply Chain Failures — Skill Revision Document (project-independent)

Purpose: itemize A03 under three headings (symptoms — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: this document is not tied to any specific project. "Covered" = a skill's current instructions address this A03 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — A03:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: the A02 document's O3 item ("patch management") was handed off to this category — B2/O7 here close that handoff.

## 0. Items excluded from scope (will not be implemented)

A03 is much more infra/vendor/org-process-heavy than A01/A02. The following items are pure tool-operation, external-service-subscription, or org-policy matters, so they're excluded from scope:

| # | Item | Why out of scope |
|---|---|---|
| B4 | No change management / supply-chain change tracking | Org process, not code |
| B5 | Missing hardening via access control and least privilege | **A01's domain** — not reopened here as a separate item (cross-reference, already covered by `audit-security`'s A01 section) |
| B6 | No separation of duty (single person code → production) | Org/HR policy, not code |
| B8 | Patching delayed / monthly-quarterly under change control | A time/process metric, not measurable in code |
| O3 | Continuously inventory with tools like Dependency Track/Check, retire.js | Tool setup/operation — `audit-security`'s A03 section already **replaces** the need for this inventory, doesn't produce a separate revision as its own item |
| O4 | Watch CVE/NVD/OSV, automate, subscribe | External service subscription, not code |
| O8 | Keep CI/CD, IDE, dev tooling regularly updated | Tool maintenance, not code |
| O9 | Staged rollout / canary deployment | Deploy strategy, not code |
| O11 | Harden code repo, dev workstation, build server/CI/CD, artifacts, IaC (MFA, IAM, signed builds) | Pure infra/access management |
| S1 | SolarWinds-type — vendor/build process compromised, malicious code injected | Happens at the vendor/registry level, very limited code-side mitigation |
| S2 | Bybit-type — conditional/hidden malicious behavior in third-party software | Same reasoning |
| S3 | Shai-Hulud-type — self-propagating package-registry worm | Same reasoning |

Conditional (covered only if the repo has a CI/CD config file — `.github/workflows`, `.gitlab-ci.yml`, etc.):

| # | Item | Condition |
|---|---|---|
| B3 | No regular vulnerability scanning/security-bulletin tracking | `audit-security` can check "is there a dependency-scan step in CI," it can't perform the scan itself |
| B10 | CI/CD security weaker than the system it builds | Reviewable if a CI/CD config file is in the repo |
| O10 | Monitor CI/CD settings, code repo, sandbox, IDE, SBOM tooling, logs, SaaS integrations, artifact/container registry changes | Same — `code-review` flags it only if a CI/CD config diff is visible |

## 1. Symptoms (in scope: 4/10, conditional: 2/10)

| # | A03 symptom | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| B1 | Component versions (direct + transitive) not tracked | ❌ | ✅ | `audit-security` (new A03 section) | Produces a direct+transitive inventory from dependency files (package.json/go.mod/requirements.txt/lockfile) |
| B2 | Software/OS/DBMS/library vulnerable, unsupported, or outdated | ❌ | ✅ | `audit-security` (new A03 section) | Flags every component in the inventory against known-vulnerability/EOL status |
| B7 | Components from untrusted sources can affect production | ❌ | ✅ | `audit-security` (new A03 section), `code-review` | Audit: scans dependency sources (official registry vs. git URL/fork, unpinned). code-review: flags the source of a newly added dependency in the diff |
| B9 | Compatibility of updated libraries not tested | ❌ | ✅ | `implement` | Tied to a rule of running the existing test suite after a dependency update/addition |
| B3 (conditional) | No regular vulnerability scanning/bulletin tracking | ❌ | 🟡 | `audit-security` (new A03 section) | Checks whether CI/CD config has a dependency-scan step (only if the repo has CI config) |
| B10 (conditional) | Weak CI/CD security | ❌ | 🟡 | `code-review`, `audit-security` | Only reviewed if a CI/CD config file appears in the diff/repo |

## 2. Preventions (in scope: 4/11, conditional: 1/11)

| # | A03 prevention | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| O1 | Generate/maintain a central SBOM; track transitive dependencies too | ❌ | ✅ | `audit-security` (new A03 section) | Audits whether SBOM/lockfile practice exists; feeds the same inventory as B1 |
| O2 | Remove unused dependencies/features | ❌ | ✅ | `audit-security` (new A03 section) | Detects unused dependencies via code scanning |
| O5 | Only pull signed packages from trusted/official sources | ❌ | ✅ | `implement`, `code-review` | implement: official/signed-source rule when adding a new dependency. code-review: flags a diff's untrusted-source dependency (same check as B7) |
| O6 | Choose versions deliberately, update when needed | ❌ | ✅ | `implement` | Deliberate (not just latest) version-choice decision when adding a new dependency |
| O7 | Alternative or virtual patch for unmaintained libraries | ❌ | ✅ | `audit-security` (new A03 section) | Flags unmaintained/abandoned libraries in the inventory; the prevention counterpart to B2 |
| O10 (conditional) | Monitor CI/CD settings, artifact/registry changes | ❌ | 🟡 | `code-review` | Flagged only if a CI/CD config file changes in the diff |

## 3. Attack scenarios (in scope: 1/4)

| # | A03 scenario template (real-world example) | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| S4 | A known-vulnerable component stays unpatched → RCE (Struts/Log4Shell-type) | ❌ | ✅ | `audit-security` (new A03 section) | The concretized form of B2+O7; closed by the same revision |

S1-S3 (SolarWinds/Bybit/Shai-Hulud-type) out of scope — see §0.

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Symptoms (10) | 4/10 in scope + 2/10 conditional | 0 | 4/4 (+ 2 conditional) |
| Preventions (11) | 4/11 in scope + 1/11 conditional | 0 | 4/4 (+ 1 conditional) |
| Attack scenarios (4) | 1/4 | 0 | 1/1 |

Before: none of A03's items were covered by the local skill set. After: 9/9 in-scope items + 3 conditional items are closed. Compared to A01 (full coverage) and A02 (13/19), A03 is the narrowest-scope category — most items stay at the vendor/registry/org-process level.

## 5. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented — 55529e1) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A03 section**, alongside the existing A01/A02 sections) | Direct+transitive dependency inventory, known-vulnerability/EOL check, untrusted-source scan, unused-dependency detection, unmaintained-library flagging, SBOM/lockfile-practice check, (conditional) CI/CD dependency-scan-step check | B1, B2, B3(conditional), B7, B9(partial), B10(conditional), O1, O2, O7, O10(conditional), S4 |
| P0 | `code-review` | A third sub-section inside the existing **Security** axis: **Dependency (OWASP A03)** — gate: diff modifies a dependency file (package.json/go.mod/requirements.txt/lockfile) or CI/CD config. Checks: (a) does the new dependency come from an official/trusted source (not a git-URL/fork/unpinned one)? (b) is the version pinned? (c, conditional) does the CI/CD config change open a new security risk (permission expansion, secret exposure)? | B7, B10(conditional), O5, O10(conditional) |
| P1 | `implement` | "Dependency hygiene (OWASP A03)" section: only official/signed sources + deliberate version choice when adding a new dependency; run the existing test suite after a dependency update/addition (compatibility regression) | B9, O5, O6 |

`tdd`, `to-spec`, `domain-modeling`, `improve-codebase-architecture` are untouched for this category — A03 has no test-discipline dimension (`tdd` already carries the "full test suite" rule, tying it to `implement` is enough), no spec-actor dimension, no domain-ownership dimension, and no module-depth dimension.

## 6. Two scopes: diff gate vs. project-wide scan (same pattern as A01/A02)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Dependency sub-section) | diff (new hole) | every PR | `skills/code-review` | implemented (55529e1) |
| `audit-security` (A03 section, alongside the existing A01/A02 sections) | whole project (existing hole) | pre-release / periodic | `skills/audit-security` | implemented (55529e1) |

`code-review`'s Dependency sub-section: runs when the diff touches a dependency file or (conditionally) CI/CD config; checks source trustworthiness + version pinning. Doesn't see out-of-diff holes.

`audit-security`'s A03 section: inventories the entire dependency tree (direct+transitive), flags vulnerability/EOL/unmaintained/untrusted-source status, audits SBOM practice. Doesn't look at a diff.

## 7. Notes

- This category was deliberately kept narrow — the real strong defenses (SBOM tooling, CVE-feed automation, signed builds, staged rollout) are tool/infra/org investments outside this skill set. The skill set only catches the "code-visible" part here (dependency file + CI/CD config diff).
- Once the A04 (Cryptographic Failures) analysis begins, the overlap between signed-package/build verification and key management will be revisited.

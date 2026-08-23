# OWASP A04:2025 — Cryptographic Failures — Skill Revision Document (project-independent)

Purpose: itemize A04 under three headings (control questions/symptoms — preventions — attack scenarios), and for each item, derive whether the current skill set covers it and which revision needs to be added to which skill.

Scope rule: this document is not tied to any specific project. "Covered" = a skill's current instructions address this A04 item as a behavior. Revisions are written into `skills/<skill>/SKILL.md`.

Source: `owasp top 10.pdf` (OWASP Top 10:2025, Turkish reference) — A04:2025 section.

Status legend: ✅ covered · 🟡 indirect/weak/conditional · ❌ not covered

Columns: Before = current skill state · After = state once the revision is applied

Context: unlike A03, A04 is **broadly in scope** — almost all of it is code-visible crypto anti-patterns (weak algorithms, ECB, hardcoded keys, MD5/SHA1, weak randomness — most of it grep-able/statically visible).

Cross-references (the PDF's own table): A04 ↔ A07 (password storage — A07 hands this off to A04), A04 ↔ A01 (JWT/cookie), A04 ↔ A02 (hardcoded secret / security-header overlap).

## 0. Out-of-scope and conditional items

Out of scope (will not be implemented):

| # | Item | Why out of scope |
|---|---|---|
| KS10 | Side-channel leaks like padding oracles | Very hard to detect statically, an implementation subtlety — can't be reduced to a behavior rule |
| O13 | Prepare for PQC (high-risk systems by end of 2030) | Forward-looking strategy/planning, produces no code-level action right now |

Conditional/partial (the code-visible part is covered, the infra/operational part is not):

| # | Item | Condition / boundary |
|---|---|---|
| KS2 | Default/weak/reused keys, missing key rotation | Key reuse + hardcoded keys are code-visible (covered); **key rotation** is operational/infra (out of scope) |
| KS11 | Can the algorithm be downgraded/bypassed | Covered if TLS min-version config is visible; general protocol-downgrade analysis is out of scope |
| O1 | Classify/label data; store keys in an HSM | **Data classification** is covered in `to-spec` as a spec-time decision; **HSM** is pure infra (out of scope) |
| O3 | Don't over-retain sensitive data; tokenization/truncation | Data minimization is covered in `to-spec` as a design decision; tokenization infra is out of scope |
| O4 | Encrypt at rest + key management | App-level encryption is code-visible (covered); DB/disk-level encryption and key-management infra are out of scope |

## 1. Control questions / symptoms (in scope: 8/11, conditional: 2/11)

| # | A04 symptom | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| KS1 | Use of old/weak algorithms-protocols | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect weak/deprecated algorithms and protocols (code + config) |
| KS3 | Are keys committed to the repo? | ❌ | ✅ | `audit-security` (A04), `code-review` | Scan for committed crypto keys/material. **Overlaps with A02 O7's secret scan** — A04 focuses on crypto material specifically, cross-ref |
| KS4 | Is HTTPS/HSTS not mandatory? | 🟡 (HSTS header is in A02) | ✅ | `audit-security` (A04), `implement` | Mandatory HTTPS (http→https redirect) + HSTS. **Overlaps with A02 B8** — cross-ref |
| KS5 | Is the certificate/trust chain verified? | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect the disabled-cert-verification anti-pattern (`rejectUnauthorized:false`, `verify=False`) |
| KS6 | IV reuse / ECB mode / plain encryption instead of authenticated encryption | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect crypto-API misuse |
| KS7 | Is a password used directly as a key (no KDF)? | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect missing KDF in password-derived keys |
| KS8 | Weak randomness instead of a CSPRNG | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect weak randomness (e.g. `Math.random()`) in a security context |
| KS9 | Deprecated hashes like MD5/SHA1 | ❌ | ✅ | `audit-security` (A04), `code-review`, `implement` | Detect deprecated hash functions (grep-able) |
| KS2 (conditional) | Default/weak/reused keys | ❌ | 🟡 | `audit-security` (A04), `code-review` | Hardcoded/reused keys in code; rotation out of scope |
| KS11 (conditional) | Algorithm downgrade/bypass | ❌ | 🟡 | `audit-security` (A04) | If TLS min-version config is visible |

## 2. Preventions (in scope: 9/13, conditional: 3/13)

| # | A04 prevention | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| O2 | Use trusted crypto libraries (don't roll your own crypto) | ❌ | ✅ | `implement`, `code-review` | "Don't roll your own crypto" rule; hand-written crypto is flagged |
| O5 | TLS 1.2+ in transit, forward secrecy, drop CBC, HSTS | ❌ | ✅ | `audit-security` (A04), `implement` | TLS config hardening (if config is in the repo) |
| O6 | Disable caching on sensitive responses | ❌ | ✅ | `code-review`, `audit-security` (A04) | `Cache-Control: no-store` check. **Overlaps with A02 headers** — cross-ref |
| O7 | Don't use unencrypted protocols like FTP/STARTTLS | ❌ | ✅ | `code-review`, `audit-security` (A04), `implement` | Detect `ftp://`, `http://` for sensitive data |
| O8 | Passwords: Argon2/scrypt/PBKDF2 (legacy bcrypt) | ❌ | ✅ | `implement`, `audit-security` (A04) | Strong password-hash scheme. **Shared boundary with A07 — A07 hands off password storage to A04** |
| O9 | IV appropriate for the mode, never reused per key | ❌ | ✅ | `implement`, `code-review` | IV-usage rule |
| O10 | Use authenticated encryption | ❌ | ✅ | `implement`, `code-review` | AEAD (GCM/ChaCha20-Poly1305) preference rule |
| O11 | Keys generated cryptographically at random, passwords via KDF | ❌ | ✅ | `implement`, `code-review` | Key-generation + KDF rule |
| O12 | Avoid deprecated functions/modes/padding (MD5, SHA1, CBC, PKCS#1 v1.5) | ❌ | ✅ | `implement`, `code-review`, `audit-security` (A04) | Deprecated-primitive avoidance rule |
| O1 (conditional) | Classify/label data | ❌ | 🟡 | `to-spec` | Sensitive-data classification at spec time; HSM out of scope |
| O3 (conditional) | Don't over-retain sensitive data; tokenization | ❌ | 🟡 | `to-spec` | Data-minimization spec decision |
| O4 (conditional) | Encrypt at rest + key management | ❌ | 🟡 | `audit-security` (A04) | If app-level encryption is visible; DB/disk infra out of scope |

## 3. Attack scenarios (in scope: 2/2)

| # | A04 scenario template | Before | After | Skill to revise | What the revision does |
|---|---|---|---|---|---|
| S1 | TLS not mandatory/weak → downgrade + session-cookie theft → session hijack | ❌ | ✅ | `audit-security` (A04), `implement` | TLS enforcement + secure/HttpOnly cookie flag; the concretized form of KS4/O5 |
| S2 | Unsalted/simple-hash password DB → cracked via rainbow table/GPU | ❌ | ✅ | `implement`, `audit-security` (A04) | The concretized form of O8/KS9; salted strong KDF |

## 4. Summary: coverage table (before → after)

| Heading | In scope/total | Before | After |
|---|---|---|---|
| Control questions (11) | 8/11 in scope + 2/11 conditional | 0 | 8/8 (+ 2 conditional) |
| Preventions (13) | 9/13 in scope + 3/13 conditional | 0 | 9/9 (+ 3 conditional) |
| Attack scenarios (2) | 2/2 | 0 | 2/2 |

Before: none of A04's items were covered by the local skill set. After: 19/19 in-scope items + 5 conditional items are closed. A04, alongside A01, is one of the broadest-scope categories — crypto is almost entirely code-visible.

## 5. Skill revisions (project-independent)

| Priority | Skill | Behavior added (implemented) | Items closed |
|---|---|---|---|
| P0 | `audit-security` (new **A04 section**, alongside the existing A01/A02/A03 sections) | Weak/deprecated algorithm (MD5/SHA1/DES/RC4/ECB/CBC-without-MAC) grep, committed key/material scan, weak randomness in a security context, missing KDF, disabled cert verification, password-hash-scheme inventory, TLS/cleartext-protocol config, sensitive-response caching | KS1, KS2(conditional), KS3, KS4, KS5, KS6, KS7, KS8, KS9, KS11(conditional), O4(conditional), O5, O6, O12, S1, S2 |
| P0 | `code-review` | Security axis → fourth sub-section **Cryptography (OWASP A04)** — gate: diff touches a crypto API call, hashing, TLS/cert config, secret/key, or password handling. Checks: (a) is it hand-written crypto (instead of a vetted library)? (b) a deprecated algorithm/hash/mode (MD5/SHA1/DES/ECB)? (c) a strong KDF for passwords, or a plain/unsalted hash? (d) a CSPRNG for tokens/keys, or `Math.random()`? (e) is cert verification disabled? (f) a newly committed key/secret? (g) authenticated encryption, or plain? | KS1, KS3, KS5, KS6, KS7, KS8, KS9, O2, O6, O7, O9, O10, O11, O12 |
| P1 | `implement` | New **"Cryptography (OWASP A04)"** section: use a vetted library (don't roll your own crypto), a strong KDF for passwords (Argon2/scrypt/PBKDF2, legacy bcrypt), a CSPRNG for tokens/keys, authenticated encryption, no deprecated primitives, no committed keys, verify certs, enforce TLS for sensitive transit + secure cookies | KS1, KS4, KS5, KS6, KS7, KS8, KS9, O2, O5, O7, O8, O9, O10, O11, O12, S1, S2 |
| P1 | `to-spec` | New **"Sensitive Data"** section (parallel to how Access Control was added for A01): classify which data the feature handles is sensitive (PII/credential/financial/health/trade-secret) at spec time; write transit/rest protection and data-minimization (don't over-retain) expectations for each. State explicitly if there's no sensitive data | O1(conditional), O3(conditional) |

`tdd`, `domain-modeling`, `improve-codebase-architecture` are untouched in this category — crypto doesn't fit a general red-test discipline (tdd), data classification sits more cleanly in `to-spec` than `domain-modeling`, and A04 has no module-depth dimension.

## 6. Two scopes: diff gate vs. project-wide scan (same pattern as A01/A02/A03)

| Skill | Scope | Trigger | Location | Status |
|---|---|---|---|---|
| `code-review` (Security axis → Cryptography sub-section) | diff (new hole) | every PR | `skills/code-review` | implemented |
| `audit-security` (A04 section, alongside the existing A01/A02/A03 sections) | whole project (existing hole) | pre-release / periodic | `skills/audit-security` | implemented |

`code-review`'s Cryptography sub-section: runs when the diff touches a crypto API/hashing/TLS-cert config/secret/password handling. Doesn't see out-of-diff holes.

`audit-security`'s A04 section: scans the entire codebase for weak algorithms / committed keys / weak randomness / missing KDF / disabled cert verification / password-hash scheme / TLS-cleartext config. Doesn't look at a diff.

## 7. Notes

- **Overlap management:** KS3 (committed keys) overlaps with A02 O7's secret scan, KS4/O6 (HSTS + caching headers) overlaps with A02 B8's header inventory. Since `audit-security` is a single file, these aren't re-scanned — the A04 section focuses on findings specific to crypto material, general secret/header scanning stays in the A02 section (cross-reference note).
- **A07 boundary:** password storage/hashing (O8, S2) is shared with A07 (Authentication Failures). Once the A07 analysis begins, it will be confirmed that A07 hands this item off to A04 (per the PDF: "For plaintext/weak-hash password storage, see A04") — the password-hashing rule's true owner is the A04 section, and A07 carries a forward-reference.

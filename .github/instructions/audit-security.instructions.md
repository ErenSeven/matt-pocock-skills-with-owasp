---
applyTo: "**"
description: "Audit a whole project for security holes against the OWASP Top 10 — a coverage inventory to run before a release, when onboarding, or periodically (not per-PR). Unlike code-review, which gates a single diff, this scans the entire codebase for holes that already exist across Broken Access Control (missing authz, IDOR/BOLA, privilege escalation), Security Misconfiguration (default credentials, leaked stack traces, missing headers, CORS), Software Supply Chain (vulnerable or unpinned dependencies), Cryptographic Failures (weak algorithms, committed keys, unsalted password hashing, disabled cert validation), Injection (SQL/NoSQL/OS-command/ORM/LDAP/template), Authentication Failures (session fixation, unverified JWT claims, user enumeration), Integrity Failures (insecure deserialization), Logging Failures (unlogged events, secrets in logs), and Exceptional Conditions (swallowed errors, fail-open, non-atomic ops). Produces a per-category inventory. For a test-coverage audit use audit-test instead."
---

<!-- Auto-generated from skills/audit-security/SKILL.md by scripts/generate-copilot.ps1. Do not edit by hand. -->


Project-wide **security audit** mapped to the OWASP Top 10, organized as one section per category. This is the *inventory* companion to `code-review`:

- `code-review` = **diff gate** — catches new holes a PR introduces (its Security axis covers the same categories as a fast, diff-scoped check).
- `audit-security` (this skill) = **whole-project sweep** — finds holes that already exist, regardless of when they were introduced.

Run this periodically or before a release, not on every change. It reads the whole codebase, not a diff. Run whichever category sections are relevant — all of them for a full audit, or a subset if the user asks for one category specifically.

## A01 — Broken Access Control

### 1. Enumerate the attack surface

Find every externally reachable entry point — the things an attacker can call:

- HTTP routes / controllers / handlers (all verbs: GET/POST/PUT/PATCH/DELETE).
- Background/job/webhook/queue consumers that act on caller-supplied data.
- GraphQL resolvers, gRPC methods, server actions, RPC endpoints.
- Any admin/privileged surface (dashboards, internal tools, feature flags).

Produce one row per entry point. Do not stop at the first framework you recognize — sweep for all of them.

### 2. Classify each entry point

For every row, determine and record:

| Column | What to answer |
|---|---|
| **Auth required?** | Public by design, or must the caller be authenticated? |
| **Authz gate?** | Is there an explicit authorization check (role/permission/ownership) before the action runs? |
| **Ownership scoped?** | If it returns/edits a record, is it filtered to the caller's *own* resources — or can any id be passed (IDOR/BOLA)? |
| **Id source** | Does ownership come from the **auth token/session** (good) or from a **client-supplied param/body** (dangerous)? |
| **Fail mode** | On auth/authz error does it fail **closed** (deny/401/403) or **open** (proceed)? |
| **Privilege level** | Does it correctly separate anonymous / user / admin? Any vertical-escalation path? |

### 3. Check the global posture (A01 preventions)

Beyond per-endpoint, verify project-wide controls once:

- **Deny by default** — is access denied unless explicitly allowed, or allowed unless explicitly blocked?
- **Centralized enforcement** — is authz applied in one reusable place (a seam/middleware/policy), or copy-pasted per handler (easy to forget → new endpoints leak)?
- **CORS** — does it fall back to `*`/AllowAnyOrigin anywhere, especially in production config?
- **Rate limiting** — are sensitive/expensive endpoints throttled?
- **Session invalidation** — logout invalidates server-side session; stateless tokens are short-lived + verifiable.
- **Access-failure logging** — are denied-authz attempts logged and alertable?

### 4. Report — A01 coverage inventory

Output two parts:

**A. Endpoint table** — every entry point from step 1 with its step-2 classification, most-risky first. Flag each risky row with the A01 sub-theme it maps to (IDOR/BOLA, missing function-level authz, privilege escalation, fail-open, forced browsing).

**B. Posture checklist** — the step-3 items as ✅ / 🟡 / ❌ with a one-line evidence note each.

End with: count of endpoints with **no authz gate**, count with **client-supplied ownership id**, and the single highest-risk finding. Recommend a prioritized fix order — centralization gaps (which cause many endpoint gaps) before individual endpoints.

## A02 — Security Misconfiguration

### 1. Sweep for unnecessary surface

Find anything exposed that shouldn't be, in production:

- Sample/demo/example applications or routes bundled with a framework and never removed.
- Debug endpoints, test-only routes, or admin tooling reachable without a feature flag.
- Unused accounts, default users, or seeded test credentials still present.

### 2. Scan for default and hardcoded credentials

- Grep for default usernames/passwords (`admin`/`admin`, framework-shipped defaults, seed-data credentials).
- Grep for hardcoded static secrets/API keys/connection strings committed directly in source or config (not sourced from env/secret-store).

### 3. Check error handling

- Is there a single, centralized error handler, or is error handling ad-hoc per endpoint?
- Does any error path return a stack trace, internal exception message, or framework/version banner to the client?

### 4. Check server/framework/library/DB configuration against secure defaults

- Any insecure legacy mode/flag left on "for backward compatibility"?
- Are server, framework, and DB settings at their hardened values (not shipped defaults)?
- Directory listing disabled; no backup/`.git`/config files served from web roots?

### 5. Check security headers

Inventory the security-relevant HTTP response headers actually sent (e.g. `Strict-Transport-Security`, `X-Content-Type-Options`, `Content-Security-Policy`, `X-Frame-Options`) versus what the framework/server is capable of sending. Note anywhere a header-setting middleware exists but is disabled or bypassed for a subset of routes.

### 6. Report — A02 coverage inventory

Output:

**A. Findings table** — one row per finding (unnecessary surface / default credential / hardcoded secret / leaked stack trace / insecure legacy config / missing header), with file/line evidence and the A02 sub-theme it maps to.

**B. Header inventory** — headers sent vs. expected, ✅ / 🟡 / ❌ per header.

End with: count of findings by category, and the single highest-risk finding (a live default credential or an exposed sample app with admin access outranks a missing header).

## A03 — Software Supply Chain Failures

This section is deliberately narrow: the strongest supply-chain defenses (SBOM tooling, CVE-feed automation, signed builds, staged rollout, separation of duty) live in tooling/infra/org process outside this skill set. Here you audit only what is visible **in the repo** — dependency manifests and, if present, CI/CD config.

### 1. Inventory the dependency tree

Find every dependency manifest and lockfile (e.g. `package.json`/`package-lock.json`, `go.mod`/`go.sum`, `requirements.txt`/`poetry.lock`, `pom.xml`, `Cargo.toml`/`Cargo.lock`, `Gemfile.lock`). Produce an inventory of components, distinguishing **direct** from **transitive** where the lockfile makes it visible. If there is no lockfile, flag that first — an unpinned tree can't be audited reliably.

### 2. Flag each component

For each component, note what the repo lets you determine:

| Column | What to answer |
|---|---|
| **Source** | Official registry (good) vs. a git URL / fork / vendored blob / unknown origin (dangerous)? |
| **Pinned?** | Is the version pinned (exact/lockfile) or floating (`^`, `*`, `latest`)? |
| **Known-vulnerable / EOL** | Does the pinned version match a known CVE or an end-of-life/unsupported release? (Judgement from version alone — you are not running a scanner.) |
| **Maintained?** | Is the library abandoned/unmaintained (a virtual-patch or replacement candidate)? |

### 3. Check supply-chain posture

- **SBOM / lockfile practice** — is there a committed lockfile and, ideally, an SBOM? Are transitive deps tracked, not just top-level?
- **Unused dependencies** — any declared dependency not actually imported anywhere (removable surface)?
- **CI/CD dependency scanning (conditional — only if CI/CD config exists in-repo)** — is there a dependency-scan / audit step in the pipeline? Note its presence/absence; you check that the step exists, you don't run the scan yourself.

### 4. Report — A03 coverage inventory

Output:

**A. Component table** — the step-2 classification, most-risky first (known-vulnerable + unpinned + untrusted-source at the top). Flag each risky row with the A03 sub-theme (known-vulnerable component, untrusted source, unpinned version, abandoned library).

**B. Posture checklist** — SBOM/lockfile present, transitive tracking, unused-deps, CI scan step, as ✅ / 🟡 / ❌ with a one-line note each.

End with: count of known-vulnerable and untrusted-source components, and the single highest-risk finding (a known-RCE component like a Log4Shell-era release outranks an unpinned-but-current one).

## A04 — Cryptographic Failures

Cryptography is highly code-visible, so this section scans the whole codebase for the anti-patterns below. Deliberately excluded: side-channel/padding-oracle detection (not statically reliable) and post-quantum readiness (a forward-looking strategy, not a code-level finding). Key **rotation**, HSM storage, and disk/DB-level encryption-at-rest are infra concerns outside this scan; app-level crypto usage is in.

**Overlap note:** committed secrets and security headers are also scanned by the A02 section. Don't re-report them here — the A04 section focuses on cryptographic material and cryptographic misuse specifically. Password storage (below) is the canonical home for the A07 authentication section's "weak password hashing" item; A07 cross-references here rather than duplicating.

### 1. Scan for weak algorithms and primitives

Grep the codebase for deprecated/broken cryptographic primitives in use:

- Hashes: MD5, SHA-1 used for security (integrity/signatures/password hashing).
- Ciphers/modes: DES/3DES, RC4, AES-ECB, CBC without an authenticated MAC, PKCS#1 v1.5.
- Note each with file/line and whether it's in a security-relevant path (a non-security checksum use of MD5 is a lower-severity judgement call).

### 2. Check key & secret handling

- **Committed key material** — private keys, symmetric keys, or crypto secrets checked into the repo (`.pem`, `BEGIN PRIVATE KEY`, hardcoded key literals). (Cross-check with the A02 secret scan; report crypto material here.)
- **Password-derived keys** — is a password used directly as an encryption key with no KDF?
- **Reused/default keys** — the same key hardcoded across environments, or a shipped default key.

### 3. Check randomness

Is security-sensitive randomness (tokens, IDs, salts, IVs, nonces) drawn from a CSPRNG, or from a weak/predictable source (`Math.random()`, `rand()`, time-seeded PRNG)?

### 4. Check password storage

What scheme hashes passwords at rest? Strong: Argon2, scrypt, PBKDF2-HMAC-SHA-512 (bcrypt acceptable as legacy). Weak/broken: plain, unsalted, single-round SHA/MD5. This is the A04↔A07 shared item — owned here.

### 5. Check transit & encryption usage

- **TLS** — is HTTPS enforced (http→https redirect), and does any in-repo TLS config set a weak floor (< TLS 1.2), keep CBC ciphers, or omit forward secrecy? Is HSTS sent? (HSTS presence overlaps the A02 header inventory — note, don't double-count.)
- **Cleartext protocols** — any `ftp://`, or `http://` used for sensitive data.
- **Certificate validation** — anywhere validation is disabled (`rejectUnauthorized: false`, `verify=False`, custom trust-all callbacks).
- **Encryption misuse** — reused/static IVs, ECB mode, or plain (non-authenticated) encryption where authenticated encryption (AEAD) is warranted.
- **Sensitive-response caching** — do responses carrying sensitive data set `Cache-Control: no-store`?

### 6. Report — A04 coverage inventory

**A. Findings table** — one row per finding (weak primitive / committed key / weak randomness / missing-KDF / weak password hash / disabled cert validation / cleartext transit / encryption misuse), file/line evidence, mapped A04 sub-theme.

End with: count of findings by sub-theme, and the single highest-risk finding (a committed private key or plaintext/unsalted password store outranks a missing `no-store` header).

## A05 — Injection

Injection is highly code-visible — the classic static-analysis category. This section sweeps for **injection sinks**: places where untrusted input reaches an interpreter (DB, shell, template engine, LDAP, expression evaluator, browser) as code rather than data. Excluded: CI/CD SAST/DAST/IAST tooling (infra), and LLM prompt injection (a separate list, LLM01 — not this Top 10).

### 1. Find the sinks, per interpreter

Sweep for each injection type and record the untrusted-input path into it:

| Type | What to grep for |
|---|---|
| **SQL** | String-concatenated / interpolated queries instead of parameterized statements/prepared queries. |
| **NoSQL** | Query objects built from raw request data (e.g. `$where`, operator injection). |
| **OS command** | `exec`, `system`, `popen`, `eval`, backticks, `child_process` with user-influenced arguments. |
| **ORM / HQL** | Raw / dynamic ORM queries (`.raw(`, HQL/JPQL string building) carrying untrusted data. |
| **LDAP** | Filters built by concatenation from user input. |
| **EL / OGNL / template** | Server-side template or expression evaluation of user-controlled strings (SSTI). |
| **Browser / XSS** | User data rendered into HTML/JS without context-aware output encoding (auto-escaping disabled, `dangerouslySetInnerHTML`, `innerHTML`, unescaped interpolation). |

### 2. Classify each sink

For each, note: is input **separated from command** (parameterized / safe API / ORM binding), or **concatenated**? Is there **positive server-side validation** upstream? For output, is there **context-aware encoding**? A parameterized query with bound values is safe; a concatenated one is a finding.

### 3. Report — A05 coverage inventory

**A. Sink table** — one row per injection sink, most-risky first (concatenated SQL/command with reachable user input at the top). Columns: interpreter type, file/line, separated-or-concatenated, validation present?, mapped A05 sub-theme.

End with: count of sinks by type, count of concatenated (unsafe) vs parameterized, and the single highest-risk finding (a reachable SQL or OS-command concatenation outranks a reflected-XSS in an admin-only view).

## A07 — Authentication Failures

Audit the authentication and session machinery. **Overlap note:** password *hashing* is owned by the A04 section (report it there), default credentials by the A02 section — this section covers authentication-flow and session mechanics, cross-referencing those rather than re-scanning. Session invalidation / rate-limit / access-failure logging also appear in the A01 posture; here they're examined through the authentication lens (login, lockout, enumeration).

### 1. Check session management

- **Session fixation** — is the session identifier regenerated on login (a new high-entropy id issued), or is the pre-login id reused?
- **Session id exposure** — is the session id ever carried in a URL or a hidden form field instead of a secure cookie?
- **Cookie flags** — are session cookies `Secure` + `HttpOnly` (+ `SameSite` where applicable)?
- **Invalidation** — does logout invalidate the server-side session, and are idle + absolute timeouts enforced? For SSO, does logout propagate (single logout)?

### 2. Check token validation

For JWT / bearer tokens: are signature, `aud`, `iss`, and expiry all asserted before the token is trusted — or is any of them skipped?

### 3. Check the login surface

- **Brute-force resistance** — is there a failed-login lockout/delay, plus logging of failed attempts?
- **Enumeration** — do "unknown user" and "wrong password" return the *same* error (and similar timing), so accounts can't be enumerated?
- **Password policy** — are weak/known/breached passwords rejected (a weak-password list), and is the policy NIST-800-63b-aligned (crucially, *not* forcing periodic rotation or arbitrary composition rules)?
- **Credential recovery** — does recovery avoid security-question-based flows (banned by NIST/OWASP; see A06)?

### 4. Report — A07 coverage inventory

**A. Findings table** — one row per finding (session fixation / session-id exposure / weak cookie flags / missing invalidation / unverified token claim / missing brute-force lockout / enumeration-friendly error / weak password policy / insecure recovery), file/line evidence, mapped A07 sub-theme.

End with: count of findings by sub-theme, and the single highest-risk finding (session fixation or an unverified JWT signature outranks a missing `SameSite` flag). Cross-reference the A04 password-hashing finding if present.

## A08 — Software or Data Integrity Failures

Narrow section: the app-visible core of integrity failures. **Overlap note:** trust of a package's *source* is owned by the A03 section, CI/CD pipeline security by the A02 section — this section covers integrity of *data and code the app itself deserializes, includes, or downloads*, cross-referencing those rather than re-scanning.

### 1. Scan for insecure deserialization

Grep for deserializers fed untrusted/client-supplied data:

- Python `pickle.load`/`loads`, `yaml.load` (without `SafeLoader`), `Marshal`.
- Java `ObjectInputStream.readObject` (watch for `rO0`-prefixed base64 = serialized Java on the wire).
- PHP `unserialize`, Ruby `Marshal.load`, .NET `BinaryFormatter`/`NetDataContractSerializer`.
- Note whether the input can originate from a client/request/message, and whether any integrity check (signature/HMAC) precedes it.

### 2. Check external resource integrity

- **Subresource Integrity** — do external/CDN `<script>` and `<link>` tags carry an `integrity` (SRI) attribute + `crossorigin`, or are they trusted blindly?
- **In-app download / auto-update** — if the app downloads code/data or self-updates, is the artifact's signature or checksum verified before use? Unverified download-and-execute is a finding.

### 3. Report — A08 coverage inventory

**A. Findings table** — one row per finding (insecure deserialization of untrusted data / missing SRI / unverified download or update), file/line evidence, and whether untrusted input reaches it.

End with: count of findings by sub-theme, and the single highest-risk finding (deserialization of client-supplied data outranks a missing SRI on a first-party script). Cross-reference the A03 source-trust findings if present.

## A09 — Security Logging & Alerting Failures

This section covers the **code side** of logging: what gets logged, what must not, and whether log data is safe. The alerting/monitoring pipeline (SIEM, SOC, thresholds, escalation, playbooks, incident response, log aggregation/backup) is ops/infra and out of scope. This section is the cross-cutting owner of logging — the A01 "access-failure logging" and A07 "failed-login logging" items are instances of it.

### 1. Check security-event coverage

Are the security-relevant events logged, consistently, with enough context (who / what / when / outcome) to be useful for forensics?

- Authentication outcomes (success *and* failure), access-control denials, input-validation rejections.
- High-value / state-changing actions (money movement, permission changes, data export).
- Every security control's decision — logged whether it allowed or denied.

Flag security-sensitive entry points that perform none of this logging.

### 2. Check for sensitive data in logs

- Are secrets, tokens, passwords, full card numbers, or PII/PHI being written to logs? (Logs are a frequently-overlooked leak channel — cross-reference A04.)
- Is log output ever exposed to the end user (leaking internal detail — cross-reference A01)?

### 3. Check log injection

Is untrusted/user-supplied data written into log messages without encoding? Unencoded newlines/control chars let an attacker forge or split log entries (log injection).

### 4. Report — A09 coverage inventory

**A. Findings table** — one row per finding (unlogged security event / sensitive data in logs / log exposed to user / log injection), file/line evidence, mapped A09 sub-theme.

End with: count of findings by sub-theme, and the single highest-risk finding (a password or full-PAN written to logs outranks a single unlogged low-value action). Note: exception handling and fail-closed/rollback belong to A10, not here.

## A10 — Mishandling of Exceptional Conditions

Audit how the code behaves when things go wrong — errors, failures, and unexpected states. This is where the exception-handling / fail-closed / rollback concerns deferred from A09 land. **Overlap note:** leaking error detail to the user is owned by the A02 section (stack traces), logging the error by the A09 section — this section covers whether the error is *handled correctly* (caught, failed closed, rolled back, resources released).

### 1. Check error handling

- **Swallowed errors** — empty/`catch {}` blocks, exceptions caught and ignored, or errors handled only at a top-level catch-all instead of where they occur and can be dealt with meaningfully.
- **Fail mode** — on an unexpected error, does the code fail **closed** (deny/abort safely) or fail **open** (proceed as if nothing happened)?

### 2. Check atomicity of multi-step operations

For any multi-step / stateful operation (especially money movement, or write-then-write sequences): if it fails partway (network drop, crash), is the whole thing rolled back (transaction / compensating action), or can it leave partial state (double-spend, account drain)? For retryable operations, is it idempotent so a retry can't double-apply?

### 3. Check resource handling and limits

- **Resource cleanup** — are files, connections, locks, and handles released on *all* paths including the error path (`finally` / `using` / `defer` / context manager)? An error path that leaks resources is a DoS vector.
- **Throttling / quotas** — are expensive or unbounded operations rate-limited / quota'd, so a flood can't exhaust memory/CPU/connections (or run up a cloud bill)? (Rate-limit for *availability*; the A01 posture covers rate-limit for *authorization abuse* — note, don't double-count.)

### 4. Report — A10 coverage inventory

**A. Findings table** — one row per finding (swallowed error / fail-open / non-atomic multi-step operation / missing idempotency / leaked resource on error path / missing throttle), file/line evidence, mapped A10 sub-theme.

End with: count of findings by sub-theme, and the single highest-risk finding (a non-atomic money-movement flow that can double-spend outranks a single unclosed file handle).

## Notes

- This skill **reports**, it does not fix. Fixes go through `implement` + `tdd` (write the failing "unauthorized → denied" test first for A01; for A02, add a regression check appropriate to the finding — e.g. a header-presence test; for A03, upgrade/replace the flagged dependency and re-run the suite to catch compatibility regressions; for A04, replace the weak primitive/scheme and, where feasible, add a test pinning the expected algorithm — e.g. that stored password hashes use the strong KDF; for A05, switch the sink to a parameterized/safe API and add an injection-payload test — malicious input treated as data, not executed; for A07, fix the session/token/login mechanic and add a test — e.g. that the session id changes on login, or that unknown-user and wrong-password return the same error; for A08, switch to a safe deserializer / add SRI / verify the signature, and add a test that a tampered payload or artifact is rejected; for A09, add the missing security-event log / redact the sensitive field / encode the logged input; for A10, handle the error at origin / fail closed / make the operation atomic / release the resource, and add a test that the failure path behaves — e.g. a mid-transaction failure leaves no partial state).
- A finding here is a **judgement call** grounded in evidence — cite the file/line and quote the relevant code. Do not assert a gap you can't point at.
- Framework-agnostic: infer the auth mechanism, error-handling pattern, and config layout from the code rather than assuming one stack.
- New OWASP categories are added as new top-level sections here (mirroring how `code-review`'s Security axis grows by sub-section, not by new skill) — see the OWASP revision docs under `docs/owasp/` for the source analysis behind each section.

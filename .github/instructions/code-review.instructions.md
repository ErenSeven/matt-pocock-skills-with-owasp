---
applyTo: "**"
description: "Review the changes since a fixed point (commit, branch, tag, or merge-base) along three axes — Standards (does the code follow this repo's documented coding standards?), Spec (does the code match what the originating issue/spec asked for?), and Security (does the diff introduce an OWASP hole — Broken Access Control, Security Misconfiguration, Software Supply Chain, Cryptographic Failures, Injection, Authentication Failures, Integrity Failures, Logging Failures, Exceptional Conditions, and other categories as they're added). Runs the reviews in parallel sub-agents and reports them side by side. Use when the user wants to review a branch, a PR, work-in-progress changes, or asks to "review since X"."
---

<!-- Auto-generated from skills/code-review/SKILL.md by scripts/generate-copilot.ps1. Do not edit by hand. -->


Three-axis review of the diff between `HEAD` and a fixed point the user supplies:

- **Standards** — does the code conform to this repo's documented coding standards?
- **Spec** — does the code faithfully implement the originating issue / spec?
- **Security** — does the diff introduce an OWASP hole? A single axis covering multiple OWASP Top 10 categories (see the sub-sections below), kept as one sub-agent so the review doesn't grow one parallel agent per category. This is a **diff gate** (new holes only). For whole-project sweeps of *existing* holes, use the separate `audit-security` skill instead.

Each axis runs as a **parallel sub-agent** so they don't pollute each other's context, then this skill aggregates their findings. The Security axis only runs when the diff touches a surface one of its sub-sections cares about (see each sub-section's own gate below); if none apply, skip the axis entirely and note "no security-relevant surface in diff".

The issue tracker should have been provided to you — run `/setup-matt-pocock-skills` if `docs/agents/issue-tracker.md` is missing.

## Process

### 1. Pin the fixed point

Whatever the user said is the fixed point — a commit SHA, branch name, tag, `main`, `HEAD~5`, etc. If they didn't specify one, ask for it.

Capture the diff command once: `git diff <fixed-point>...HEAD` (three-dot, so the comparison is against the merge-base). Also note the list of commits via `git log <fixed-point>..HEAD --oneline`.

Before going further, confirm the fixed point resolves (`git rev-parse <fixed-point>`) and the diff is non-empty. A bad ref or empty diff should fail here — not inside two parallel sub-agents.

### 2. Identify the spec source

Look for the originating spec, in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`, etc.) — fetch via the workflow in `docs/agents/issue-tracker.md`.
2. A path the user passed as an argument.
3. A spec file under `docs/`, `specs/`, or `.scratch/` matching the branch name or feature.
4. If nothing is found, ask the user where the spec is. If they say there isn't one, the **Spec** sub-agent will skip and report "no spec available".

### 3. Identify the standards sources

Anything in the repo that documents how code should be written, such as `CODING_STANDARDS.md` or `CONTRIBUTING.md`.

On top of whatever the repo documents, the Standards axis always carries the **smell baseline** below — a fixed set of Fowler code smells (_Refactoring_, ch.3) that applies even when a repo documents nothing. Two rules bind it:

- **The repo overrides.** A documented repo standard always wins; where it endorses something the baseline would flag, suppress the smell.
- **Always a judgement call.** Each smell is a labelled heuristic ("possible Feature Envy"), never a hard violation — and, like any standard here, skip anything tooling already enforces.

Each smell reads *what it is* → *how to fix*; match it against the diff:

- **Mysterious Name** — a function, variable, or type whose name doesn't reveal what it does or holds. → rename it; if no honest name comes, the design's murky.
- **Duplicated Code** — the same logic shape appears in more than one hunk or file in the change. → extract the shared shape, call it from both.
- **Feature Envy** — a method that reaches into another object's data more than its own. → move the method onto the data it envies.
- **Data Clumps** — the same few fields or params keep travelling together (a type wanting to be born). → bundle them into one type, pass that.
- **Primitive Obsession** — a primitive or string standing in for a domain concept that deserves its own type. → give the concept its own small type.
- **Repeated Switches** — the same `switch`/`if`-cascade on the same type recurs across the change. → replace with polymorphism, or one map both sites share.
- **Shotgun Surgery** — one logical change forces scattered edits across many files in the diff. → gather what changes together into one module.
- **Divergent Change** — one file or module is edited for several unrelated reasons. → split so each module changes for one reason.
- **Speculative Generality** — abstraction, parameters, or hooks added for needs the spec doesn't have. → delete it; inline back until a real need shows.
- **Message Chains** — long `a.b().c().d()` navigation the caller shouldn't depend on. → hide the walk behind one method on the first object.
- **Middle Man** — a class or function that mostly just delegates onward. → cut it, call the real target direct.
- **Refused Bequest** — a subclass or implementer that ignores or overrides most of what it inherits. → drop the inheritance, use composition.

### 4. Spawn the sub-agents in parallel

**Standards sub-agent prompt** — include:

- The full diff command and commit list.
- The list of standards-source files you found in step 3, **plus the smell baseline from step 3** pasted in full — the sub-agent has no other access to it.
- The brief: "Report — per file/hunk where relevant — (a) every place the diff violates a documented standard: cite the standard (file + the rule); and (b) any baseline smell you spot: name it and quote the hunk. Distinguish hard violations from judgement calls — documented-standard breaches can be hard, but baseline smells are always judgement calls, and a documented repo standard overrides the baseline. Skip anything tooling enforces. Under 400 words."

**Spec sub-agent prompt** — include:

- The diff command and commit list.
- The path or fetched contents of the spec.
- The brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that wasn't asked for (scope creep); (c) requirements that look implemented but where the implementation looks wrong. Quote the spec line for each finding. Under 400 words."

If the spec is missing, skip the Spec sub-agent and note this in the final report.

**Security sub-agent prompt** — include:

- The diff command and commit list.
- The brief: "Review the diff for OWASP holes it introduces, across the sub-sections below. Only run a sub-section if its own gate condition matches the diff; skip it otherwise and say so. Report findings grouped under each sub-section's own heading. Quote the hunk for each finding and name the sub-theme. Findings are judgement calls grounded in the diff — don't assert a hole you can't point at. Under 500 words total.

  **Access Control (OWASP A01)** — gate: diff touches an entry point (route/handler/controller/resolver/server-action) or an auth/CORS/rate-limit config. Check: (a) is there an authorization gate before the action runs? (b) if it returns/edits a record, is it scoped to the caller's *own* resources, or can any id be passed (IDOR/BOLA)? (c) does ownership come from the auth token/session, not a client-supplied param/body? (d) does it fail closed (deny/401/403) on auth error, not open? (e) any privilege-escalation path (anonymous→user, user→admin)? Also flag any CORS fallback to AllowAnyOrigin, removed rate-limit, or newly served static/metadata files.

  **Security Config (OWASP A02)** — gate: diff adds/changes an entry point, error handler, middleware, or server/framework/DB config file. Check: (a) is a debug/sample/test endpoint being added or left enabled? (b) is a default or hardcoded static credential/secret being introduced? (c) does an error path return a stack trace or other internal detail to the client instead of a generic message? (d) is a legacy insecure mode/flag being re-enabled or left on for backward compatibility? (e) is a security-header middleware being weakened or removed?

  **Dependency (OWASP A03)** — gate: diff changes a dependency manifest/lockfile (package.json, go.mod, requirements.txt, pom.xml, Cargo.toml, a lockfile, etc.) or a CI/CD config file. Check: (a) is a newly added dependency coming from an official/trusted registry, not a git URL, fork, or vendored blob of unknown origin? (b) is the version pinned rather than floating (`^`/`*`/`latest`)? (c) does the added/changed version match a known-vulnerable or EOL release you recognize? (d) for a CI/CD config change: does it widen permissions, expose a secret, or pull build steps from an untrusted source? This sub-section flags supply-chain surface the diff introduces; the whole dependency-tree audit lives in `audit-security`.

  **Cryptography (OWASP A04)** — gate: diff touches a crypto/hashing call, TLS or certificate config, a key/secret, or password handling. Check: (a) is it hand-rolling crypto instead of a vetted library? (b) a deprecated primitive (MD5/SHA-1 for security, DES/3DES/RC4, AES-ECB, CBC without a MAC, PKCS#1 v1.5)? (c) passwords hashed with a strong KDF (Argon2/scrypt/PBKDF2, bcrypt as legacy), not plain/unsalted/fast-hash? (d) security-sensitive randomness (tokens, salts, IVs) from a CSPRNG, not `Math.random()`/time-seeded? (e) certificate validation being disabled (`rejectUnauthorized: false`, `verify=False`)? (f) a key/secret being committed? (g) reused/static IV, or plain encryption where authenticated encryption (AEAD) is warranted? Committed-secret and security-header findings overlap A02 — attribute each finding to one sub-section, don't double-report.

  **Injection (OWASP A05)** — gate: diff touches a DB query, OS/shell command execution, template render, LDAP/expression evaluation, or otherwise passes user-controlled input into an interpreter (including HTML/JS output to a browser). Check: (a) are queries parameterized / built with a safe API / ORM binding, not string-concatenated? (b) is an OS command constructed from user input? (c) does a raw ORM/HQL query carry untrusted data? (d) is browser-bound output context-aware encoded, or is auto-escaping bypassed (`innerHTML`, `dangerouslySetInnerHTML`, raw interpolation)? (e) is there positive server-side validation on the untrusted input? Remember table/column names can't be escaped — they must come from an allowlist, not user input.

  **Authentication (OWASP A07)** — gate: diff touches login/logout, session management, JWT/token handling, password policy, or credential recovery. Check: (a) is the session id regenerated on login (not reused → fixation)? (b) is a session id carried in a URL or hidden field instead of a secure cookie, and are cookies `Secure`/`HttpOnly`? (c) for JWT/bearer tokens, are signature, `aud`, `iss`, and expiry all verified before trust? (d) do auth failures return a generic message (`unknown user` and `wrong password` identical → no enumeration)? (e) is a security-question-based recovery flow being introduced (banned — see A06)? (f) is logout/timeout actually invalidating the session? Password *hashing* belongs to the Cryptography sub-section — attribute it there, not here.

  **Integrity (OWASP A08)** — gate: diff deserializes data, includes an external/CDN script or stylesheet, or downloads/auto-updates code or data. Check: (a) is untrusted/client-supplied data passed to an unsafe deserializer (`pickle`, `yaml.load`, Java `readObject`, PHP `unserialize`, .NET `BinaryFormatter`) without an integrity check? (b) does an external `<script>`/`<link>` include a Subresource Integrity (`integrity`) attribute, or is it trusted blindly? (c) is a downloaded/self-updated artifact's signature or checksum verified before use? (d) is client-supplied serialized state trusted without a signature/HMAC? Package *source* trust belongs to the Dependency sub-section — attribute it there, not here.

  **Logging (OWASP A09)** — gate: diff adds/changes a security-sensitive action (auth, an access-control decision, a high-value/state-changing operation) or adds/changes a log statement. This is a **light** check — most logging gaps are about what's *missing* and don't fully show in a diff. Check: (a) does a newly added sensitive action emit an audit log (who/what/outcome), or is it silent? (b) does a new/changed log statement write a secret, token, password, full card number, or PII? (c) is untrusted/user input written into a log message without encoding (log injection)? Exception handling and fail-closed belong to the Exceptional Conditions sub-section (A10), not here.

  **Exceptional Conditions (OWASP A10)** — gate: diff adds/changes error handling, a multi-step/transactional operation, resource acquisition (files/connections/locks), or an expensive/unbounded operation. Check: (a) is an error swallowed (empty catch) or handled only at a far-off top level, instead of at origin? (b) does it fail **closed** on error, not open? (c) is a multi-step operation atomic (rolled back on failure), and idempotent if retryable — no double-spend/partial state? (d) are resources released on the error path too (`finally`/`using`/`defer`)? (e) is an expensive operation throttled/quota'd against exhaustion? Leaking error detail to the user belongs to Security Config (A02), and logging the error to Logging (A09) — attribute those there."

If the diff touches no surface any sub-section's gate cares about, skip this sub-agent entirely and note "no security-relevant surface in diff".

### 5. Aggregate

Present the reports under `## Standards`, `## Spec`, and `## Security` headings (with `### Access Control` / `### Security Config` / `### Dependency` / `### Cryptography` / `### Injection` / `### Authentication` / `### Integrity` / `### Logging` / `### Exceptional Conditions` sub-headings inside Security, whichever sub-sections actually ran), verbatim or lightly cleaned. Do **not** merge or rerank findings across axes — the axes are deliberately separate (see _Why separate axes_).

End with a one-line summary: total findings per axis, and the worst issue _within each axis_ (if any). Don't pick a single winner across axes — that's the reranking the separation exists to prevent.

## Why separate axes

A change can pass one axis and fail another:

- Code that follows every standard but implements the wrong thing → **Standards pass, Spec fail.**
- Code that does exactly what the issue asked but breaks the project's conventions → **Spec pass, Standards fail.**
- Code that is clean and matches the spec but exposes another user's data or leaks a stack trace → **Standards + Spec pass, Security fail.**

Reporting them separately stops one axis from masking another. Security is the *diff gate* — new OWASP holes only, across whichever sub-sections its gate matches; the whole-project counterpart is `audit-security` (its A01–A05 and A07–A10 sections mirror these sub-sections; A06 Insecure Design has no diff gate — it lives in the design-time skills `to-spec` and `domain-modeling`).

## Why Security stays one axis, not one-per-category

Each OWASP Top 10 category could in principle be its own parallel sub-agent, the way Access Control was added for A01. That doesn't scale: most categories are irrelevant to most diffs, and spawning one idle sub-agent per category per PR wastes agents and produces a report nobody reads in full. Instead, new categories are added as a **sub-section inside Security** — its own gate condition and its own checklist, but sharing the one sub-agent and one report heading. Only split a category out into its own top-level axis if it needs isolation Security can't give it (e.g. a fundamentally different kind of reviewer, or a gate so broad it would run on nearly every diff and drown out the others).

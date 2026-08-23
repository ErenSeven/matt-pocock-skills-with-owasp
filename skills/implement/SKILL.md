---
name: implement
description: "Implement a piece of work based on a spec or set of tickets."
disable-model-invocation: true
---

Implement the work described by the user in the spec or tickets.

Use /tdd where possible, at pre-agreed seams.

## Authorization by default (OWASP A01)

When the work adds or changes an entry point (route, handler, controller, resolver, server action, job that acts on caller data):

- **Gate before action.** Enforce authorization before returning data or performing a side effect — never after. Default to deny unless the spec grants access.
- **Scope to the owner.** If it reads or edits a record, filter to the caller's own resources. Derive ownership from the authenticated identity (token/session), never from a client-supplied id, param, or body field — those can be forged.
- **Fail closed.** On any auth/authz error, deny (401/403) and stop; do not fall through to the happy path.
- **Log denials.** Emit a log for failed authorization attempts so abuse is observable.
- **Reuse the existing gate.** Apply the project's established authorization seam/middleware rather than hand-rolling a check per endpoint; if none exists and checks are being copy-pasted, flag it (candidate for `/improve-codebase-architecture`).

Follow the `/tdd` rule: the first test for a protected seam is the denial case (unauthorized / other-owner → refused).

## Secure defaults (OWASP A02)

When the work adds or changes error handling, a server/route/framework setup, or credentials/secrets:

- **Never leak internals to the client.** Route errors through a single, centralized handler. A client-facing error response is a generic message — never a stack trace, internal exception message, or framework/version banner.
- **Security headers on by default.** When standing up a new server or route group, apply the project's standard security headers (e.g. `Strict-Transport-Security`, `X-Content-Type-Options`, `Content-Security-Policy`, `X-Frame-Options`) rather than leaving framework defaults as-is.
- **No hardcoded default credentials or static secrets.** Source credentials, API keys, and connection strings from environment/secret-store, never literals in code or config. Don't leave a seeded default account/password reachable.
- **Don't re-enable insecure legacy modes for compatibility.** If an old flag/mode is insecure, flag it instead of flipping it back on to satisfy a compatibility requirement — surface the trade-off to the user rather than silently choosing it.

## Dependency hygiene (OWASP A03)

When the work adds or upgrades a third-party dependency:

- **Trusted source only.** Pull from the official/canonical registry — not a git URL, fork, or vendored blob of unknown origin. Prefer packages that publish signatures/provenance where the ecosystem supports it.
- **Choose the version deliberately.** Don't reach for `latest` or a floating range by reflex; pick a specific, currently-supported version and pin it (lockfile). Avoid known-vulnerable or end-of-life releases.
- **Re-run the suite after a dependency change.** An upgrade can break behavior silently — run the existing tests after adding or bumping a dependency to catch compatibility regressions, not just after your own code changes.

## Cryptography (OWASP A04)

When the work touches hashing, encryption, tokens, TLS/certificates, keys, or password handling:

- **Don't roll your own crypto.** Use a vetted, current library; never hand-implement a cipher, hash construction, or protocol.
- **Hash passwords with a strong KDF.** Argon2, scrypt, or PBKDF2-HMAC-SHA-512 (bcrypt acceptable as legacy), always salted. Never plain, unsalted, or a fast hash (MD5/SHA-*) for passwords.
- **Use a CSPRNG for anything security-sensitive.** Tokens, session IDs, salts, IVs, nonces come from a cryptographically secure RNG — never `Math.random()`, `rand()`, or a time-seeded PRNG.
- **Prefer authenticated encryption.** Use AEAD (AES-GCM, ChaCha20-Poly1305) over plain modes; never AES-ECB; never reuse an IV/nonce under the same key.
- **Avoid deprecated primitives.** No MD5/SHA-1 for security, DES/3DES/RC4, or PKCS#1 v1.5.
- **Never commit keys or secrets.** Source key material from env/secret-store; don't check private keys into the repo (see also the A02 secret rule above).
- **Verify certificates.** Never disable TLS certificate validation (`rejectUnauthorized: false`, `verify=False`) outside a controlled test.
- **Protect data in transit.** Enforce HTTPS for sensitive data, mark session cookies `Secure`/`HttpOnly`, and don't use cleartext protocols (`ftp://`, `http://`) for sensitive payloads.

If the spec's **Sensitive Data** section classifies data as sensitive, treat encryption-in-transit and at-rest for that data as a requirement, not an option.

## Injection prevention (OWASP A05)

When the work passes input into an interpreter — a DB query, an OS/shell command, a template, an LDAP filter, an expression evaluator, or HTML/JS sent to a browser:

- **Separate data from command.** Use parameterized queries / prepared statements, a safe API, or ORM binding — never build a query or command by string concatenation/interpolation of user input.
- **Validate positively, server-side.** Allowlist expected shapes/values on the server; treat client-side validation as UX only. (Positive validation reduces risk but is not a substitute for separation above.)
- **Encode on output, context-aware.** For browser-bound output, rely on the framework's context-aware auto-escaping; don't bypass it (`innerHTML`, `dangerouslySetInnerHTML`, raw interpolation) with untrusted data.
- **Allowlist what can't be escaped.** Identifiers like table/column names cannot be parameterized or escaped — they must come from a fixed server-side allowlist, never from request data.

Follow the `/tdd` rule: for a seam that feeds an interpreter, add an injection-payload test (malicious input is treated as data, not executed).

## Authentication (OWASP A07)

When the work touches login/logout, session management, tokens, password policy, or credential recovery:

- **Don't roll your own auth.** Prefer a vetted, maintained authentication/identity/session system over hand-built login and session code (parallels "don't roll your own crypto").
- **Manage sessions server-side.** Issue a new high-entropy session id on login (defeats fixation); never carry a session id in a URL or hidden field; set cookies `Secure`/`HttpOnly` (+ `SameSite`); invalidate on logout and on idle/absolute timeout.
- **Verify tokens fully.** For JWT/bearer tokens, assert signature, `aud`, `iss`, and expiry before trusting — don't skip any.
- **Fail auth generically.** Return one message for unknown-user and wrong-password (and comparable timing) so accounts can't be enumerated.
- **Resist brute force.** Lock out or delay after repeated failures, and log failed attempts (feeds the logging/alerting layer).
- **Follow NIST 800-63b for passwords.** Reject weak/known-breached passwords; do *not* force periodic rotation or arbitrary composition rules. Prefer MFA over password complexity.
- **No security-question recovery.** Use a proper recovery flow, not knowledge-based questions (banned by NIST/OWASP; see the spec's Threat Model).

Password *hashing* is covered by the Cryptography rule above (strong KDF) — apply it for any stored credential.

## Integrity (OWASP A08)

When the work deserializes data, includes an external resource, or downloads/updates code or data:

- **Don't deserialize untrusted data unsafely.** Never feed client/request/message data to a native-object deserializer (`pickle`, `yaml.load` without `SafeLoader`, Java `readObject`, PHP `unserialize`, .NET `BinaryFormatter`). Use a safe data format (JSON) or a deserializer restricted to an allowlist of types.
- **Pin external resources with SRI.** Give external/CDN `<script>` and `<link>` tags a Subresource Integrity `integrity` hash + `crossorigin`, so a tampered CDN asset is rejected.
- **Verify what you download.** For any downloaded artifact or self-update, check a signature or checksum before using or executing it.
- **Don't trust client-supplied serialized state.** If serialized state round-trips through the client, protect it with a signature/HMAC and verify before use.

Package/dependency *source* trust is covered by the Dependency hygiene rule above (A03) — this rule is about integrity of what you deserialize, include, or download.

## Logging & alerting (OWASP A09)

When the work performs a security-relevant action (authentication, an access-control decision, a high-value or state-changing operation) or adds logging:

- **Log security events with context.** Record auth outcomes (success *and* failure), access denials, and high-value actions with who/what/when/outcome — enough to reconstruct events for forensics.
- **Never log secrets or PII.** Keep passwords, tokens, full card numbers, and PII/PHI out of logs; redact or omit them. Don't expose raw logs to end users.
- **Encode untrusted data in log messages.** Neutralize newlines/control characters in user-supplied values before logging, so entries can't be forged or split (log injection).
- **Emit an alert signal on suspicious events.** Where the app can flag a suspicious pattern (repeated auth failures, a tripped security control), emit a signal for the monitoring layer — even though the alerting pipeline itself is ops.
- **Keep the audit trail append-only in intent.** Treat security/audit records as append-only; don't offer code paths that rewrite or delete them.

Handling the exception itself — catching it, failing closed, rolling back — is covered separately (exceptional conditions); this rule is that the event must be *logged*.

## Exceptional conditions (OWASP A10)

When the work handles errors, performs a multi-step/stateful operation, acquires resources, or exposes an expensive operation:

- **Handle errors at origin, don't swallow them.** Catch where the error occurs and deal with it meaningfully; never leave an empty catch or lean on a distant top-level catch-all as the only handling.
- **Fail closed.** On an unexpected error, deny/abort safely — never proceed as if it succeeded.
- **Make multi-step operations atomic.** If a sequence can fail partway (crash, network drop), wrap it so the whole thing rolls back (transaction / compensating action) — no partial state, no double-spend. Make retryable operations idempotent so a retry can't double-apply.
- **Release resources on every path.** Free files, connections, locks, and handles in `finally`/`using`/`defer`/context managers, so the error path can't leak them (a DoS vector).
- **Bound expensive work.** Rate-limit / quota / throttle operations that could exhaust memory, CPU, connections, or a cloud budget.
- **Route through the central handler.** Send errors to one consistent handler that returns a generic message to the user (A02) and logs the event (A09).

Follow the `/tdd` rule: add an exceptional-path test (a mid-operation failure leaves no partial state and fails closed), not just the happy path.

Run typechecking regularly, single test files regularly, and the full test suite once at the end.

Once done, use /code-review to review the work.

Commit your work to the current branch.

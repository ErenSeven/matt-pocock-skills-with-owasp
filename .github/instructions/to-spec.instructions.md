---
applyTo: "**"
description: "Turn the current conversation into a spec and publish it to the project issue tracker — no interview, just synthesis of what you've already discussed."
---

<!-- Auto-generated from skills/to-spec/SKILL.md by scripts/generate-copilot.ps1. Do not edit by hand. -->


This skill takes the current conversation context and codebase understanding and produces a spec. Do NOT interview the user — just synthesize what you already know.

The issue tracker and triage label vocabulary should have been provided to you — run `/setup-matt-pocock-skills` if not.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the spec, and respect any ADRs in the area you're touching.

2. Sketch out the seams at which you're going to test the feature. Existing seams should be preferred to new ones. Use the highest seam possible. If new seams are needed, propose them at the highest point you can. The fewer seams across the codebase, the better - the ideal number is one.

Check with the user that these seams match their expectations.

3. Write the spec using the template below, then publish it to the project issue tracker. Apply the `ready-for-agent` triage label - no need for additional triage.

<spec-template>

## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a mobile bank customer, I want to see balance on my accounts, so that I can make better informed decisions about my spending
</user-story-example>

This list of user stories should be extremely extensive and cover all aspects of the feature.

## Implementation Decisions

A list of implementation decisions that were made. This can include:

- The modules that will be built/modified
- The interfaces of those modules that will be modified
- Technical clarifications from the developer
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions

Do NOT include specific file paths or code snippets. They may end up being outdated very quickly.

Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it within the relevant decision and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Access Control

Who is allowed to do what (OWASP A01 — capture authorization at spec time so it isn't an afterthought). For each actor in the user stories, state:

- **Allowed actions** — what this actor may do.
- **Forbidden actions** — what this actor must NOT do, especially reads/writes on resources they don't own.
- **Ownership boundary** — for any resource the feature exposes, whose it is and how ownership is proven (from the authenticated identity, never a client-supplied id).
- **Default posture** — access is denied unless a story explicitly grants it (deny by default).
- **Privilege levels** — anonymous vs authenticated vs elevated/admin, and any action that changes level.

If the feature has no access-controlled surface (purely public/read-only), say so explicitly rather than omitting the section.

## Sensitive Data

What data this feature handles that is sensitive (OWASP A04 — classify at spec time so encryption and minimization aren't afterthoughts). For each piece of sensitive data:

- **Classification** — is it credentials/secrets, PII, financial, health, or trade-secret data? (These carry regulatory weight — GDPR/PCI DSS/HIPAA.)
- **In transit** — must it move only over encrypted channels (TLS enforced, no cleartext protocols)?
- **At rest** — must it be encrypted or hashed when stored? Passwords in particular must use a strong KDF (Argon2/scrypt/PBKDF2, not a plain/fast hash).
- **Minimization** — is storing it necessary at all, or can it be tokenized/truncated/not-stored? Don't retain sensitive data the feature doesn't need.

If the feature handles no sensitive data, say so explicitly rather than omitting the section.

## Threat Model / Abuse Cases

How this feature could be misused (OWASP A06 — design the controls in, rather than discovering they're missing after an incident; insecure design can't be fixed by good implementation). This section is the security umbrella: the Access Control and Sensitive Data sections above are two of its outputs. For the auth, access-control, business-logic, and critical flows in this feature:

- **Abuse cases** — for each important flow, state "who would misuse this, and how": a user acting on another tenant's data, someone bypassing a business-logic limit (deposit/quota/threshold), automated/bulk abuse (scalping, scraping), an actor escalating privilege. Write these as misuse-cases alongside the user stories.
- **Acceptance criterion per abuse case** — for each, the control that defeats it (a limit enforced server-side, an ownership check, a rate limit) and how you'd know it holds.
- **Business-logic limits** — thresholds, quotas, and plausibility rules the design must enforce (not just input validation, but "does this sequence of actions make sense"). These belong in the domain model as invariants — flag them for `/domain-modeling`.
- **Banned-by-design patterns** — call out known-insecure designs so they don't get built: e.g. security-question-based credential recovery (banned by NIST/OWASP — use a proper recovery flow instead).

If the feature has no meaningful abuse surface, say so explicitly rather than omitting the section.

## Testing Decisions

A list of testing decisions that were made. Include a **denial test** for every access-controlled action from the Access Control section (unauthorized / other-owner → refused), and a **misuse-case test** for each abuse case from the Threat Model section (the abuse is blocked / the limit holds). Also include:

- A description of what makes a good test (only test external behavior, not implementation details)
- Which modules will be tested
- Prior art for the tests (i.e. similar types of tests in the codebase)

**Test levels** — decide, at spec time, *what kind* of test each part of the feature needs, so implementation doesn't default to unit-only coverage. The seams sketched in step 2 already say where; this says at what level. Prefer the highest seam that gives the confidence you need (see `tdd`'s `test-types.md` for the vocabulary):

- **Integration test** — call it out for any cross-module or module↔infrastructure contract this feature introduces (repository↔DB, controller↔service through the real pipeline). A contract only ever exercised end-to-end, or not at all, is a gap.
- **Contract test** — required when the feature depends on an independently-deployed service it doesn't control (another team's API, a third party). Name it here so the boundary isn't left to a brittle full end-to-end run.
- **End-to-end / subcutaneous test** — for each *critical* whole-path user flow (login, payment, data sync), decide whether it needs one end-to-end proof (or a subcutaneous one just below the UI, for e2e-level confidence without UI-automation fragility). Keep these few; don't specify one per story.
- **Snapshot test** — for rendered UI, note whether snapshot coverage applies and that non-deterministic values (timestamps, ids) must be stubbed.

If the feature is small enough that unit tests at the agreed seams suffice, say so explicitly rather than omitting this.

**Acceptance format (optional)** — if the business side needs to read and agree with the tests, note that the acceptance criteria should be expressed in Given-When-Then (Gherkin) form so they're executable *and* readable by non-engineers. Only call this out when a stakeholder actually needs it; the user stories above already carry the business intent otherwise.

## Out of Scope

A description of the things that are out of scope for this spec.

## Further Notes

Any further notes about the feature.

</spec-template>

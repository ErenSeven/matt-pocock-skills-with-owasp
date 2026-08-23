---
name: to-issues
description: 'Break a plan, spec, or PRD into independently-grabbable issues on the project issue tracker using tracer-bullet vertical slices. Use when the user wants to convert a plan into issues, create implementation tickets, or break down work into issues.'
argument-hint: 'Optional: issue number, URL, or path to a PRD/spec to break down'
user-invocable: true
disable-model-invocation: false
---

# To Issues

Break a plan, spec, or PRD into independently-grabbable issues using **tracer-bullet vertical slices**.

## When to Use

- User says "break this into issues", "create tickets", "convert to issues", "make implementation tickets"
- After a PRD or spec exists (in the conversation or on the issue tracker)
- When the user wants to plan work as independently-executable slices for agents or team members

## Procedure

### 1. Gather Context

Work from whatever is already in the conversation. If the user passes an issue reference (issue number, URL, or path) as an argument, fetch it from the issue tracker and read its full body and comments.

### 2. Explore the Codebase (if not already done)

Explore the repo to understand the current state of the code:
- Read the README, any ADRs in `docs/decisions/`, and domain contracts
- Identify module boundaries, naming conventions, and the tech stack
- Issue titles and descriptions **must** use the project's domain glossary and respect ADRs in the area being touched

### 3. Draft Vertical Slices

Break the plan into **tracer bullet** issues. Each issue is a thin vertical slice that cuts through ALL integration layers end-to-end — **not** a horizontal slice of one layer.

Classify each slice as:
- **HITL** (Human-In-The-Loop): requires human interaction (architectural decision, design review, external dependency)
- **AFK** (Away From Keyboard): can be implemented and merged without human interaction

**Vertical slice rules:**
- Each slice delivers a narrow but COMPLETE path through every layer (schema, API, UI, tests)
- A completed slice is demoable or verifiable on its own
- Prefer many thin slices over few thick ones
- Prefer AFK over HITL where possible

### 4. Quiz the User

Present the proposed breakdown as a numbered list. For each slice, show:

- **Title**: short descriptive name
- **Type**: HITL / AFK
- **Blocked by**: which other slices (if any) must complete first
- **User stories covered**: which user stories this addresses (if the source material has them)

Ask the user:
- Does the granularity feel right? (too coarse / too fine)
- Are the dependency relationships correct?
- Should any slices be merged or split further?
- Are the correct slices marked as HITL and AFK?

Iterate until the user approves the breakdown.

### 5. Publish Issues to the Issue Tracker

For each approved slice, publish a new issue using the template below. Publish in **dependency order** (blockers first) so real issue identifiers can be referenced in the "Blocked by" field.

Apply the correct triage label unless instructed otherwise. Do **not** close or modify any parent issue.

**Issue body template:**

```markdown
## Parent

A reference to the parent issue on the issue tracker (if the source was an existing issue, otherwise omit this section).

## What to Build

A concise description of this vertical slice. Describe the end-to-end behavior, not layer-by-layer implementation.

Avoid specific file paths or code snippets — they go stale fast. Exception: if a prototype produced a snippet that encodes a decision more precisely than prose can (state machine, reducer, schema, type shape), inline it here and note briefly that it came from a prototype. Trim to the decision-rich parts — not a working demo, just the important bits.

## Acceptance Criteria

- [ ] Criterion 1
- [ ] Criterion 2
- [ ] Criterion 3

## Blocked by

- A reference to the blocking ticket (if any)

Or "None - can start immediately" if no blockers.
```

### 6. Report Back

After publishing, report back to the user:
- A list of all created issues with their URLs and titles
- The dependency graph (which issues block which)
- Any HITL slices that need human action before agents can proceed

## Quality Checks

Before publishing, verify:
- [ ] Every slice is independently demoable or verifiable
- [ ] No slice is purely a horizontal layer (e.g., "add all DB migrations")
- [ ] Blockers form a DAG — no circular dependencies
- [ ] Domain vocabulary from the codebase is used consistently
- [ ] ADRs in the touched area are respected
- [ ] HITL slices are minimized — each one is justified
- [ ] Triage label is applied to all AFK issues

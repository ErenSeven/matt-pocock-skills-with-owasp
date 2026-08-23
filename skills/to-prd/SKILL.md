---
name: to-prd
description: 'Turn the current conversation context into a PRD and publish it to the project issue tracker. Use when the user wants to create a PRD, document a feature, write a product requirements document, or publish a spec from the current context. Do NOT interview the user — synthesize from what is already known.'
argument-hint: 'Optional: feature name or additional context to include'
user-invocable: true
disable-model-invocation: false
---

# To PRD

## What This Skill Does

Synthesizes the current conversation context and codebase understanding into a structured PRD, then publishes it to the project issue tracker (GitHub Issues). Does **not** interview the user — it draws on what is already known.

## When to Use

- User says "create a PRD", "write a PRD", "turn this into a PRD", or "publish a spec"
- After a design discussion or implementation plan has been established
- When feature scope needs to be formalized for a team or tracker

## Procedure

### 1. Explore the Repository

If you haven't already explored the repo, do so now:
- Read the README, any ADRs in `docs/decisions/`, and domain contracts.
- Identify the existing module boundaries and naming conventions.
- Note the tech stack and any shared vocabulary used across the codebase.
- Use the project's domain glossary throughout the PRD — never invent new terms.

### 2. Sketch the Major Modules

Identify the modules that will be built or modified:
- Prefer **deep modules**: high functionality behind a simple, stable, testable interface.
- Distinguish new modules from modifications to existing ones.
- Check for opportunities to extract isolated, independently testable units.
- Present the module list to the user briefly and confirm it matches their expectations.
- Ask the user which modules they want tests written for.

### 3. Write the PRD

Use the template below. Be thorough — especially in user stories (aim for comprehensive coverage of all actors and scenarios).

```markdown
## Problem Statement

The problem that the user is facing, from the user's perspective.

## Solution

The solution to the problem, from the user's perspective.

## User Stories

A LONG, numbered list of user stories:

1. As a <actor>, I want <feature>, so that <benefit>

Cover all actors and edge cases. The list should be extensive.

## Implementation Decisions

- Modules to build/modify and their interfaces
- Architectural decisions
- Schema changes
- API contracts
- Specific interactions
- Technical clarifications

Do NOT include specific file paths or code snippets — they go stale quickly.
Exception: if a prototype snippet encodes a decision more precisely than prose (state machine, reducer, schema, type shape), inline it and note it came from a prototype. Trim to decision-rich parts only.

## Testing Decisions

- What makes a good test for this feature (test external behavior, not implementation details)
- Which modules will be tested
- Prior art: similar tests already in the codebase

## Out of Scope

Things explicitly excluded from this PRD.

## Further Notes

Any remaining notes, risks, open questions, or follow-ups.
```

### 4. Publish to Issue Tracker

- Publish the PRD as a new issue to the project issue tracker.
- Apply the `ready-for-agent` triage label.
- No additional triage or approval step is needed.
- Report the issue URL back to the user.

## Quality Checks

Before publishing, verify:
- [ ] Domain vocabulary from the codebase is used consistently throughout
- [ ] ADRs in the area being touched are respected
- [ ] User stories cover all actors (end users, admins, external systems, etc.)
- [ ] Implementation decisions reference modules, not files
- [ ] Testing decisions name specific modules and describe what "good" looks like
- [ ] Out of Scope section is explicit — avoids ambiguity about what is deferred

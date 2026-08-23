---
name: domain-modeling
description: Build and sharpen a project's domain model. Use when the user wants to pin down domain terminology or a ubiquitous language, record an architectural decision, or when another skill needs to maintain the domain model.
---

# Domain Modeling

Actively build and sharpen the project's domain model as you design. This is the *active* discipline — challenging terms, inventing edge-case scenarios, and writing the glossary and decisions down the moment they crystallise. (Merely *reading* `CONTEXT.md` for vocabulary is not this skill — that's a one-line habit any skill can do. This skill is for when you're changing the model, not just consuming it.)

## File structure

Most repos have a single context:

```
/
├── CONTEXT.md
├── docs/
│   └── adr/
│       ├── 0001-event-sourced-orders.md
│       └── 0002-postgres-for-write-model.md
└── src/
```

If a `CONTEXT-MAP.md` exists at the root, the repo has multiple contexts. The map points to where each one lives:

```
/
├── CONTEXT-MAP.md
├── docs/
│   └── adr/                          ← system-wide decisions
├── src/
│   ├── ordering/
│   │   ├── CONTEXT.md
│   │   └── docs/adr/                 ← context-specific decisions
│   └── billing/
│       ├── CONTEXT.md
│       └── docs/adr/
```

Create files lazily — only when you have something to write. If no `CONTEXT.md` exists, create one when the first term is resolved. If no `docs/adr/` exists, create it when the first ADR is needed.

## During the session

### Challenge against the glossary

When the user uses a term that conflicts with the existing language in `CONTEXT.md`, call it out immediately. "Your glossary defines 'cancellation' as X, but you seem to mean Y — which is it?"

### Sharpen fuzzy language

When the user uses vague or overloaded terms, propose a precise canonical term. "You're saying 'account' — do you mean the Customer or the User? Those are different things."

### Discuss concrete scenarios — including misuse (OWASP A06)

When domain relationships are being discussed, stress-test them with specific scenarios. Invent scenarios that probe edge cases and force the user to be precise about the boundaries between concepts.

Push this past edge-cases into **misuse-cases**: don't only ask "what's a tricky legitimate scenario," ask "how would someone *abuse* this?" Insecure design (A06) is at root a model that never considered the adversary. For each important flow, invent the abuse: someone reaching another tenant's data, someone bypassing a business-logic limit, someone driving a flow with automation or bulk actions it wasn't designed for. Each abuse the model can't rule out is a missing invariant.

Turn the ones that survive into **business-logic invariants**:

- **Limits and thresholds** — quotas, deposit/spending thresholds, rate ceilings, state-transition rules ("a Booking can't be confirmed without a deposit"). Model these as invariants, not as scattered checks — a bulk-reservation or threshold-bypass abuse is a limit that lived only in the UI, not the domain.
- **Plausibility** — "does this sequence of actions make sense?" State the rule that makes an implausible sequence a violation.

### Model ownership and roles (OWASP A01)

Access control is a domain question before it is a code question: broken-access-control (IDOR/BOLA, privilege escalation) is at root an unclear model of *who owns what* and *who may act as whom*. When a resource concept appears, pin down:

- **Ownership** — who owns this resource, and through which relationship? "A Booking belongs to the Customer who created it" is a domain invariant, not an implementation detail. Name the owner explicitly.
- **Roles / privilege levels** — which actors exist (anonymous, member, owner, admin…) and what each may do to which resources. Make the actor a named term in the glossary when it carries authorization meaning.
- **Boundaries** — can actor A ever act on actor B's resource? State the rule ("only the owner or an admin may cancel a Booking") as an invariant.
- **Tenant segregation** — if the system is multi-tenant, whose tenant does a resource belong to? Cross-tenant access is a domain invariant violation, not an edge case: state "a resource is only ever visible within its owning tenant" as a hard boundary, so isolation is modeled rather than assumed.

Capture these as glossary terms and invariants in `CONTEXT.md`; when the trade-off is load-bearing and hard to reverse, offer an ADR.

### Cross-reference with code

When the user states how something works, check whether the code agrees. If you find a contradiction, surface it: "Your code cancels entire Orders, but you just said partial cancellation is possible — which is right?"

### Update CONTEXT.md inline

When a term is resolved, update `CONTEXT.md` right there. Don't batch these up — capture them as they happen. Use the format in [CONTEXT-FORMAT.md](./CONTEXT-FORMAT.md).

`CONTEXT.md` should be totally devoid of implementation details. Do not treat `CONTEXT.md` as a spec, a scratch pad, or a repository for implementation decisions. It is a glossary and nothing else.

### Offer ADRs sparingly

Only offer to create an ADR when all three are true:

1. **Hard to reverse** — the cost of changing your mind later is meaningful
2. **Surprising without context** — a future reader will wonder "why did they do it this way?"
3. **The result of a real trade-off** — there were genuine alternatives and you picked one for specific reasons

If any of the three is missing, skip the ADR. Use the format in [ADR-FORMAT.md](./ADR-FORMAT.md).

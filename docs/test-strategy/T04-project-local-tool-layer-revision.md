# Test Strategy — Project-Local Tool Layer · Skill Revision Document

> **Purpose:** T01–T03 deliberately kept the global skills **technology-agnostic** (no tool
> names). This document closes the open end of that deliberate choice: *where* the concrete tool
> mapping (Stryker.NET, Detox, Pact, k6…) is written, and how the skills consume it.
>
> **Why a separate layer:** the rule from [USAGE.md §8](../USAGE.md) — "skill content must stay
> project-independent, no repo-specific names/paths/examples". The same split as
> `audit-access-control` staying global while a project's own `code-review` copy stays local in
> the OWASP work ([owasp-a01 §6](../owasp/owasp-a01-skill-revision.md)): **global = agnostic
> rule, local = concrete tool.**

Status legend: ✅ applied · 🟡 partial · ❌ absent · 📝 planned

---

## 1. The problem

| # | Topic | Before | After | Where |
|---|---|:--:|:--:|---|
| A1 | An agnostic test-type vocabulary exists, but the concrete tool mapping has no home | ❌ | ✅ | `docs/agents/test-tools.md` (per project) |
| A2 | Who/what scaffolds this file | ❌ | ✅ | `setup-matt-pocock-skills` (when a test skill is installed) |
| A3 | How skills find the concrete tool | ❌ | ✅ | `test-types.md` "Project-local tools" + `audit-test` reads it directly |

**Before:** T01–T03 said "projects map this in their own `.claude/skills`," but there was no concrete landing spot — the user's wish for "as much technology-inclusion as possible" had nowhere to go. **After:** closed via the `test-tools.md` mechanism + the `setup` skeleton + consumer pointers.

---

## 2. Architecture

```
GLOBAL (agnostic, ~/.claude/skills)          PROJECT-LOCAL (concrete, inside the repo)
  tdd/test-types.md  ── vocabulary ──►  docs/agents/test-tools.md
  "which type, when"                    "we use xUnit/Stryker.NET/Detox"
        ▲                                      ▲
        │ reference                            │ setup scaffolds it
  tdd · to-spec · implement            setup-matt-pocock-skills
  audit-test · code-review               (when a test skill is installed)
```

- The **global layer** never contains a tool name; only test-type/discipline names.
- The **local layer** (`docs/agents/test-tools.md`) maps those types onto the repo's actual tools/commands.
- **If the file exists**, skills name the concrete tool/command; **if it doesn't**, they stay agnostic (inferring from the environment — `package.json`/project files — rather than inventing a tool).

---

## 3. Skill revisions

| Skill | What was done | Status |
|---|---|---|
| `setup-matt-pocock-skills` | New seed `test-tools.md`; Section D (write a blank template when a test skill is installed); `### Test tools` added to the `## Agent skills` block; explore/confirm/write/done steps updated | ✅ |
| `tdd/test-types.md` | A closing "Project-local tools" section — use the concrete tool if the file exists, otherwise stay agnostic and treat the environment as the source of truth | ✅ |
| `audit-test` | A direct line next to the Vocabulary note — if `test-tools.md` exists, make the posture checks concrete (a listed discipline is *expected*, so its absence is a sharper finding) | ✅ |
| `to-spec` · `implement` | No separate edit needed — both already reference `test-types.md` and inherit the "Project-local tools" pointer | ✅ (indirect) |

**Note — why `code-review` isn't included:** the Tests axis runs on a self-contained sub-agent brief (it doesn't read files). Concrete tool knowledge is unnecessary for a diff gate — "is there a test / is it at the right level" is tool-independent. `code-review` is deliberately excluded as a consumer of `test-tools.md`.

---

## 4. Seed template — what `test-tools.md` contains

- A back-reference to the agnostic vocabulary (test-types.md) + the "stay agnostic if the file is absent" rule.
- How each skill uses this file (tdd/implement/audit-test/to-spec).
- A **blank** mapping table to fill in: Unit/Integration/Component/Contract/E2E/Snapshot/Mutation/Load/Coverage
  → Tool → Command → Notes.
- One table per stack (monorepo/multi-stack support).
- A **multi-ecosystem example** in a comment (teaches the format without tying it to one stack): .NET
  (xUnit/Stryker.NET/Pact/k6) and React Native (Jest/RNTL/Detox).
- "Keeping it honest": if a listed tool can't be found in the repo = a gap to surface, don't invent a different tool.

---

## 5. Summary

| Layer | Before | After |
|---|:--:|:--:|
| Home for the concrete tool mapping | ❌ absent | ✅ `docs/agents/test-tools.md` |
| Who scaffolds it | ❌ | ✅ `setup` (conditional) |
| Consumer pointers | ❌ | ✅ `test-types.md` + `audit-test` (to-spec/implement indirect) |

The global skills stayed project-independent (§8 preserved); technology detail is now added per repo, in one single, documented place. The user's wish for "as inclusive as possible + technology-aware" was met without breaking the agnostic core.

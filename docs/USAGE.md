# matt-pocock-skills-with-owasp — Setup & Usage Guide

This guide gets you started using the shared agent skills in this repo with **Claude Code** and **GitHub Copilot**. Skills are project-independent: the source lives here, projects *consume* it (they don't fork it).

---

## 1. Mental model (read this first)

Two targets, one source:

```
skills/<name>/SKILL.md          ← SOURCE (single source of truth, edit this by hand)
        │
        │  scripts/generate-copilot.ps1
        ▼
.github/copilot-instructions.md        ← GENERATED (Copilot index — do NOT edit by hand)
.github/instructions/*.instructions.md ← GENERATED (Copilot per-skill — do NOT edit by hand)
```

- **Claude Code** reads the `skills/` folder directly (via a link) — no generation step needed.
- **GitHub Copilot** reads the generated `.github/` output — the output ships committed in the repo, consumers never generate it themselves.

**Golden rule:** Only edit `skills/<name>/SKILL.md`. Everything under `.github/` is generated; don't touch it by hand, regenerate instead.

---

## 2. PowerShell situation (check this first)

The only "command required" step in setup is **linking** the repo into place (symlink/junction). There are three ways to do this; pick per your environment:

| Environment | Linking method | Admin/Developer Mode needed? |
|---|---|---|
| PowerShell allowed | `New-Item -ItemType SymbolicLink ...` | Required |
| **PowerShell banned** (corporate) | `cmd /c mklink /J ...` (**junction**) | **Not required** |
| Neither available | **Copy** via File Explorer | Not required (but re-copy after every update) |

This guide gives both a **PowerShell and a cmd/junction** version of every step. If PowerShell is banned, just use the cmd lines — you won't need any PowerShell command.

> **Note for contributors:** `scripts/generate-copilot.ps1` is PowerShell and is only run by whoever **edits skills** (see §8). Someone who **only consumes** skills never runs this script — the generated `.github/` output ships ready in the repo.

---

## 3. Prerequisites

| Required | For |
|---|---|
| Git | Cloning/updating the repo |
| Claude Code (CLI/IDE/desktop) | Using skills with Claude |
| GitHub Copilot (VS Code / JetBrains) | Using skills with Copilot |
| cmd.exe **or** PowerShell | Running the linking command (see §2) |

---

## 4. Step 1 — Clone the repo

```bash
git clone https://github.com/ErenSeven/matt-pocock-skills-with-owasp.git
```

From here on, `<repo-path>` means the full path of this clone (e.g. `C:\Users\<you>\source\repos\matt-pocock-skills-with-owasp`).

---

## 5. Claude Code setup

Link the `skills/` folder into the global Claude skills directory (`%USERPROFILE%\.claude\skills`) so it auto-loads in **every project**. Pick your method per §2.

> If `%USERPROFILE%\.claude\skills` **already exists** (an old copy/install), remove it first. A junction/symlink can't be created over an existing folder.

### 5.a PowerShell banned — cmd + junction (recommended corporate path)

Put this in a `.bat` file and double-click, or run it in a cmd window:

```cmd
rmdir /S /Q "%USERPROFILE%\.claude\skills"
mklink /J "%USERPROFILE%\.claude\skills" "<repo-path>\skills"
```

- `mklink /J` = **junction**; behaves like a symlink for folders, **doesn't require admin rights**.
- The `rmdir` line only removes the old copied folder — its contents already exist in the repo, nothing is lost.

### 5.b PowerShell allowed — symlink

```powershell
New-Item -ItemType SymbolicLink -Path "$HOME\.claude\skills" -Target "<repo-path>\skills"
```

### 5.c macOS / Linux

```bash
ln -s "<repo-path>/skills" "$HOME/.claude/skills"
```

### Verify

```cmd
dir "%USERPROFILE%\.claude\skills"
```

If you see `<JUNCTION>` (or symlink) next to the folder, the link is set up — it now points at the repo instead of being a copy. Then open Claude Code and try invoking a skill with `/`: `/code-review`, `/audit-security`, `/to-spec`. If you see names like `audit-security`, `code-review`, `implement`, `tdd`, `to-spec`, `domain-modeling` in the list, setup is done.

### Claude — updating

```bash
git -C "<repo-path>" pull
```

Because it's a link (junction/symlink), it's **automatically up to date** after `pull` — including newly added skills. No extra step. (Only if you chose the "copy" path in §2 do you need to re-copy after every `pull`.)

---

## 6. GitHub Copilot setup (per project)

Copilot settings are **per project**: you link the generated output into each target project's `.github/` folder. You do **not** need to run the generator script — the output ships ready in the repo.

### 6.a PowerShell banned — cmd + junction/hardlink

```cmd
mklink /H "<project>\.github\copilot-instructions.md" "<repo-path>\.github\copilot-instructions.md"
mklink /J "<project>\.github\instructions" "<repo-path>\.github\instructions"
```

- `/H` (hardlink) for the file, `/J` (junction) for the folder — neither requires admin rights.

### 6.b PowerShell allowed — symlink

```powershell
New-Item -ItemType SymbolicLink -Path "<project>\.github\copilot-instructions.md" -Target "<repo-path>\.github\copilot-instructions.md"
New-Item -ItemType SymbolicLink -Path "<project>\.github\instructions"          -Target "<repo-path>\.github\instructions"
```

### 6.c Linking unavailable — copy

Copy `<repo-path>\.github\copilot-instructions.md` and the `<repo-path>\.github\instructions\` folder into the target project's `.github/`. (In this case you need to re-copy after every update.)

### Verify

Open Copilot Chat in the project and ask a question; if Copilot's responses apply behaviors from these skills (e.g. security rules, TDD discipline), the output is loaded. `.github/copilot-instructions.md` is always loaded in full.

> ⚠️ **Copilot caveat:** Copilot has no equivalent of Claude's "invoked on demand, triggered by description" skill calling. `copilot-instructions.md` is always loaded in full, and `*.instructions.md` files apply per their `applyTo` glob (currently `**` = everything, i.e. always-on too). A large/rarely-used skill may not be well-suited to sit permanently in Copilot's context — review the generated output per skill before relying on it.

### Copilot — updating

```bash
git -C "<repo-path>" pull
```

If you linked (6.a / 6.b), the `.github/` output is automatically up to date. (If you copied — 6.c — re-copy after every `pull`.)

---

## 6.5 Starting work in a new repo — `setup-matt-pocock-skills`

Having installed the skills **globally** (§5) does not automatically configure every repo you work in. There are two separate layers:

| Layer | Lives where | Scope |
|---|---|---|
| **The skills themselves** (`skills/`) | `~/.claude/skills` junction | **Global** — install once, usable in every repo |
| **Repo configuration** (`docs/agents/*.md`, `CLAUDE.md` block) | *Inside* the repo you're working on | **Per repo** — generated by `setup-matt-pocock-skills` |

`setup-matt-pocock-skills` writes files into the **current working directory** (the target repo you're working on) — not globally. So when you pull a repo that has no config yet:

1. Open Claude Code **from the target repo's root** and run `/setup-matt-pocock-skills` — **once per repo**.
2. This writes `docs/agents/issue-tracker.md` + `triage-labels.md` + `domain.md`, and an "Agent skills" block into `CLAUDE.md`/`AGENTS.md`, into that repo.
3. After that, tracker/triage-dependent skills behave correctly.

> ⚠️ Run setup **inside the target repo** you'll work on, **not inside this skills repo**.

### Needed in every repo? — It depends

**Skills that read config** (setup required):

- `to-spec`, `to-tickets`, `to-prd`, `to-issues` — write to the issue tracker
- `triage`, `wayfinder` — tracker + triage labels
- `code-review` — pulls the spec source from the issue

> If these skills can't find the config, they'll already point you to setup ("run `/setup-matt-pocock-skills` if not provided").

**Skills that don't need config** (work without setup):

- `audit-security`, `tdd`, `implement`, `diagnosing-bugs`, `prototype`, `research`, `domain-modeling`, `codebase-design`, `grill-me`, `resolving-merge-conflicts`, `handoff`, `wizard`

Most of these read `CONTEXT.md`/ADRs if present, and lazily create them if needed otherwise. If you'll only use this group, you can skip the setup step.

**Summary:** Adding or revising skills doesn't affect setup — `setup-matt-pocock-skills` doesn't hold a skill *list*, it's roster-agnostic; it only sets up shared config. Run it once per new target repo (if you'll use tracker-dependent skills) — the rest is automatic.

---

## 7. Day-to-day usage

### 7.1 How skills are invoked

- **Claude Code:** type `/<skill-name>` (e.g. `/to-spec`, `/implement`, `/code-review main-branch`). Some skills also auto-trigger (via description matching).
- **Copilot:** no separate invocation — the rules are always active in context; just write code/ask questions normally and Copilot applies the rules.

### 7.2 Typical development flow

```
/to-spec      → Turn the conversation into a spec (with Access Control, Sensitive Data, Threat Model sections)
   ▼
/implement    → Implement the spec (secure defaults + authz + crypto + injection + auth + integrity + logging + exceptional rules built in)
   ▼
/tdd          → Write tests first (including denial, injection, misuse-case, auth, exceptional-path security tests)
   ▼
/code-review  → Review the diff along three axes: Standards / Spec / Security
```

### 7.3 Security workflow (OWASP Top 10:2025)

Two different triggers, two different skills:

| When | Skill | Scope |
|---|---|---|
| **Every PR / diff** | `/code-review <base-point>` | Catches OWASP holes **newly introduced** by the diff (Security axis, 9 sub-sections) |
| **Pre-release / periodic** | `/audit-security` | Scans **all existing** holes across the whole project, produces a per-category inventory |

- `code-review` is a **diff gate** — fast, per-PR. Its Security axis carries 9 sub-sections within a single sub-agent: Access Control (A01), Security Config (A02), Dependency (A03), Cryptography (A04), Injection (A05), Authentication (A07), Integrity (A08), Logging (A09), Exceptional Conditions (A10). Each sub-section has its own trigger condition; it doesn't run on unrelated diffs.
- `audit-security` is a **project-wide sweep** — slow, release/periodic. It carries sections for the same categories; it doesn't look at a diff, it inventories the whole codebase.
- **A06 (Insecure Design)** isn't visible at the code level — that's why it's absent from `code-review`/`audit-security`. It's handled by design-time skills: `/to-spec` (Threat Model / Abuse Cases section) and `/domain-modeling` (misuse-case, business-logic limits, tenant segregation).

**Practical recommendation:** when starting a feature, `/to-spec` (its Access Control + Sensitive Data + Threat Model sections catch security at design time) → `/implement` + `/tdd` while building → `/code-review` on the PR → `/audit-security` before release.

### 7.4 OWASP coverage map (which category, where)

| OWASP 2025 | code-review | audit-security | implement | tdd | to-spec | domain-modeling |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| A01 Broken Access Control | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| A02 Security Misconfiguration | ✅ | ✅ | ✅ | | | |
| A03 Supply Chain | ✅ | ✅ | ✅ | | | |
| A04 Cryptographic | ✅ | ✅ | ✅ | | ✅ | |
| A05 Injection | ✅ | ✅ | ✅ | ✅ | | |
| A06 Insecure Design | | | | ✅ | ✅ | ✅ |
| A07 Authentication | ✅ | ✅ | ✅ | ✅ | | |
| A08 Integrity | ✅ | ✅ | ✅ | | | |
| A09 Logging | ✅ | ✅ | ✅ | | | |
| A10 Exceptional Conditions | ✅ | ✅ | ✅ | ✅ | | |

Detailed analysis per category (what's covered/not covered and why) lives in `docs/owasp/A0X-*.md`.

### 7.5 Skill catalog (all of them, what they do)

Skills are designed as an end-to-end **flow** that turns an idea into shipped code, plus helpers around it. The `ask-matt` skill is the map of all of this (router). Below is the full list grouped by role; details for each live in its own `skills/<name>/SKILL.md`.

#### Router and setup

| Skill | What it does |
|---|---|
| `ask-matt` | The answer to "which skill should I use?" — a router that organizes skills into main-flow / on-ramp / standalone / glossary layers. |
| `setup-matt-pocock-skills` | Run **once, before other skills**. Configures the repo: issue tracker (GitHub/GitLab/local markdown/other), triage labels, domain doc layout (`CONTEXT.md` + `docs/adr/`). |

#### Main flow: idea → ship

| Skill | What it does |
|---|---|
| `grill-with-docs` | Sharpens an idea through a relentless interview; start here while inside the working directory. Leaves a lasting trace in `CONTEXT.md` and ADRs. Runs `grilling` + `domain-modeling` internally. |
| `to-spec` | Turns the conversation into a **spec** (doesn't interview, synthesizes). With Access Control (A01), Sensitive Data (A04), Threat Model/Abuse Cases (A06) sections. Publishes to the tracker. |
| `to-tickets` | Splits the spec into **tracer-bullet vertical slices**, each with a "blocked by" edge. Expand–contract for large refactors. |
| `implement` | Implements a ticket; runs `/tdd` internally, `/code-review` at the end, then commits. OWASP A01–A10 secure-default rules built in. |
| `tdd` | Red→green loop. Tests behavior through the public interface (the "seam"). Denial test first on a protected seam, injection test on an interpreter seam, exceptional-path test on the error path. |
| `code-review` | Reviews a diff along **three axes** in parallel: Standards (coding standards + Fowler smells), Spec, Security (newly introduced OWASP holes). Axes are reported separately. |

#### On-ramps (starting points that generate work)

| Skill | What it does |
|---|---|
| `triage` | Runs incoming issues/PRs *you didn't create* through a state machine (bug/enhancement × needs-triage/needs-info/ready-for-agent/ready-for-human/wontfix). Writes an agent-ready brief. |
| `diagnosing-bugs` | A 6-phase discipline for hard bugs. Its core rule: forbids forming a theory before building a single **tight feedback loop** that reliably turns red on this bug. Then minimize → hypothesis → instrumentation → fix + regression → post-mortem. |
| `wayfinder` | For **huge, foggy** work that doesn't fit in one session. Draws a shared map from decision tickets in the tracker, resolves them one at a time (produces **decisions**, not a deliverable). The "fog of war" concept. Hands off to `to-spec` once the fog clears. |

#### Codebase health (maintenance)

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Scans the code, finds **deepening opportunities** that turn shallow modules into deep ones, presents them in a visual HTML report (before/after), processes the chosen one with `grilling`. |
| `codebase-design` | The deep-module design vocabulary: module, interface, depth, seam, adapter, leverage, locality, and principles like the deletion test. `tdd` and architecture skills speak this language. |
| `domain-modeling` | Sharpens a project's *domain* language: challenges vague terms, models ownership/roles (A01), turns misuse into invariants (A06), writes irreversible decisions to ADRs. Keeps `CONTEXT.md` clean. |

#### Standalone (used on their own, outside the flow)

| Skill | What it does |
|---|---|
| `grilling` | The interview *primitive* itself. Models decisions as a "design tree," asks frontier questions in rounds. Finding facts is the agent's job, deciding is the user's. |
| `grill-me` | The **stateless** form of `grilling`; used when there's no working directory (a plan, writing, a design), leaves no files behind. |
| `prototype` | **Throwaway** code that answers a single design question. Two branches: "is the logic/state model right?" (single HTML file) or "what should it look like?" (URL-param UI variations). |
| `research` | Launches a **background agent**: investigates a question against primary sources, leaves a cited Markdown file behind; you keep working in the meantime. |
| `handoff` | Compresses the conversation into a handoff document (OS temp) for a fresh agent to pick up. Adds a "suggested skills" section, redacts secrets. |
| `to-questionnaire` | Generates a questionnaire for work whose answers live **in someone else's head**. Interrogates you not about the topic, but about the **submission** (who, what's needed). |
| `wizard` | Generates an interactive bash script for steps only **a human** can do (infrastructure, credentials/CI secrets, an unfamiliar dashboard, a one-off migration). |
| `resolving-merge-conflicts` | Resolves an in-progress merge/rebase conflict hunk by hunk, per intent, runs checks, finishes the operation. Never runs `--abort`. |
| `wait-what` | A corrective for when the last message didn't land; the agent restates what it said in plain English (ASD-STE100) and `CONTEXT.md`'s vocabulary. |
| `teach` | Teaches a topic across **multiple sessions**; uses the current directory as a stateful learning workshop (MISSION, lessons, reference, learning records). |
| `writing-for-agents` | A reference for writing documents agents read (skills, AGENTS.md, pointed-at docs): context pointers, progressive disclosure, leading words, pruning. |
| `audit-security` | The **whole-project** counterpart of `code-review`. Scans the entire codebase (not a diff) per OWASP category, produces a coverage inventory. Pre-release / periodic. |

> **Note — skill pairs that do the same job:** `to-tickets`↔`to-issues` and `to-spec`↔`to-prd` are two variants of the same job. `ask-matt` refers to the `to-spec`/`to-tickets` pair as the main flow; `to-issues`/`to-prd` are alternate/older variants. Decide as a team which one you consider "official."

### 7.6 Flow map (idea → ship)

```
                    ON-RAMPS
  incoming issues ──► /triage ─────────────┐
  something's broken ► /diagnosing-bugs ────┤
  huge foggy work ──► /wayfinder ───────────┤ (produces decisions, then to /to-spec)
                                            ▼
  IDEA ──► /grill-with-docs ──► /to-spec ──► /to-tickets ──► /implement ──► ship
              (grilling +            │            │              │  └─ /tdd
               domain-modeling)      │            │              └─ /code-review
                                     ▼            ▼
                            (design question?)  (multi-session build?)
                             /prototype            ↑ yes: split into tickets
                             bridge via /handoff    no: /implement in the same context

  MAINTENANCE: /improve-codebase-architecture  •  GLOSSARY: /codebase-design, /domain-modeling
  SECURITY: /code-review on every PR  •  /audit-security before release
```

---

## 8. Contributing — editing a skill

> This section is **only for people editing skill content**. If you're just consuming skills, skip it.

1. Only edit the relevant `skills/<name>/SKILL.md` file.
2. Regenerate the Copilot output:
   ```powershell
   ./scripts/generate-copilot.ps1
   ```
   (The script also cleans up stale output for deleted/renamed skills.)

   **PowerShell banned:** you can't run this script; two options — (a) generate it on a machine where PowerShell is allowed and commit that, or (b) port the script to a portable language (Node/Python). Don't skip generation and hand-edit `.github/` output instead — that breaks the sync.
3. Commit both the `skills/` changes and the generated `.github/` changes together — they must stay in sync (out-of-sync generated output is exactly the problem this repo is set up to prevent).
4. Push; every linked project gets updated.

**Rule:** skill content must stay project-independent — no repo-specific names/paths/examples. Project-specific skills stay in that project's own `.claude/skills/` folder, not here.

---

## 9. Troubleshooting

| Symptom | Fix |
|---|---|
| PowerShell banned, can't create a symlink | §5.a / §6.a — use `cmd /c mklink /J` (folder) and `mklink /H` (file); no admin rights needed. |
| `mklink` says "already exists" | The target folder/file already exists. Remove it first: `rmdir /S /Q "<path>"` for a folder, `del "<path>"` for a file, then re-run `mklink`. |
| `mklink /D` says "access denied" | `/D` (symlink) requires admin; use `/J` (junction) instead — no permission required. |
| Skills don't show up in Claude | Check that `%USERPROFILE%\.claude\skills` is really a link pointing at the repo: `dir "%USERPROFILE%\.claude\skills"` output should show `<JUNCTION>`/`<SYMLINK>` and the correct target. |
| Copilot isn't applying the rules | Check that the project's `.github/copilot-instructions.md` exists and is non-empty; make sure Copilot is enabled in that repo. |
| `.github/` output is out of sync with the skills | Run `./scripts/generate-copilot.ps1` and commit the change — the output may have been hand-edited (it shouldn't be). If PowerShell isn't available, follow the path in §8. |
| Old content still showing after an update | You used the "copy" path (§5/§6) — not a link. Re-copy, or switch to a junction via §5.a/§6.a (no more re-copying needed). |

---

## 10. Summary — quick start (no PowerShell / corporate)

```cmd
:: 1. Clone
git clone https://github.com/ErenSeven/matt-pocock-skills-with-owasp.git

:: 2. Link into Claude Code (cmd + junction)
rmdir /S /Q "%USERPROFILE%\.claude\skills"
mklink /J "%USERPROFILE%\.claude\skills" "<repo-path>\skills"

:: 3. (For Copilot, per project) link the .github output
mklink /H "<project>\.github\copilot-instructions.md" "<repo-path>\.github\copilot-instructions.md"
mklink /J "<project>\.github\instructions" "<repo-path>\.github\instructions"

:: 4. Update (a single command, since it's a link)
git -C "<repo-path>" pull

:: 5. Use: /to-spec -> /implement -> /tdd -> /code-review ; /audit-security before release
```

If PowerShell is allowed, see §5.b and §6.b for the symlink equivalents of the same steps.

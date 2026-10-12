# matt-pocock-skills-with-owasp

Tool-agnostic agent skills, shared across projects. Source of truth for skill content lives here — project repos consume it, they don't fork it.

> **Attribution:** most of the skills in this repo originate from [Matt Pocock](https://github.com/mattpocock)'s [skills](https://github.com/mattpocock/skills) repo (MIT licensed) — all credit for their original design goes to him. `audit-security` is original to this repo. The `docs/owasp/` decision logs document the OWASP Top 10 coverage revisions layered on top of his originals. See [LICENSE](LICENSE).

Targets: **Claude Code** and **GitHub Copilot**.

📘 Full setup + usage guide: [docs/USAGE.md](docs/USAGE.md)

## Structure

```
skills/<name>/SKILL.md       source content (one skill per folder, Claude Code format)
scripts/generate-copilot.ps1 builds Copilot output from skills/
.github/copilot-instructions.md      generated index (do not edit by hand)
.github/instructions/*.instructions.md generated per-skill files (do not edit by hand)
docs/owasp/                  OWASP Top 10 skill-coverage decision logs (see below)
docs/test-strategy/          test-strategy decision logs
docs/performance/            performance decision logs
```

Edit only under `skills/`. Everything under `.github/` in this repo is generated — regenerate after any change:

```powershell
./scripts/generate-copilot.ps1
```

## Using this in a project

The only step that needs a command is linking this repo into place. Pick per your environment:

- **PowerShell allowed:** `New-Item -ItemType SymbolicLink ...` (needs Admin/Developer Mode).
- **PowerShell banned (corporate):** `cmd /c mklink /J ...` (**junction**, no Admin needed) — this is what the examples below use.
- **Neither:** copy the folders manually (re-copy after every update).

Full walkthrough with both variants: [docs/USAGE.md](docs/USAGE.md).

**Claude Code** — link this repo's `skills/` into the global skills directory so every project picks it up. If `%USERPROFILE%\.claude\skills` already exists, remove it first (`rmdir /S /Q`) — a junction can't be created over an existing folder:

```cmd
rmdir /S /Q "%USERPROFILE%\.claude\skills"
mklink /J "%USERPROFILE%\.claude\skills" "<path-to-this-repo>\skills"
```

If a project needs its own project-only skills alongside these, keep those in the project's own `.claude/skills/` — Claude Code merges global and project-local skills.

**Copilot** — link the generated `.github/copilot-instructions.md` and `.github/instructions/` into the target project's `.github/` (per project). No need to run the generator; the output ships committed:

```cmd
mklink /H "<project>\.github\copilot-instructions.md" "<path-to-this-repo>\.github\copilot-instructions.md"
mklink /J "<project>\.github\instructions" "<path-to-this-repo>\.github\instructions"
```

**Updating** — because these are links, a single `git -C "<path-to-this-repo>" pull` refreshes both Claude and Copilot; no re-linking needed (re-copy only if you chose the copy path). The `generate-copilot.ps1` generator is PowerShell and is only for **editing** skills (see the guide), not for consuming them.

## Docs

The skill set is organised around three quality concerns, each covered at spec time, write time, diff time (`code-review`) and as a whole-project audit. The per-item analysis behind every revision (sources, before/after coverage, which skill got what) lives in the revision docs:

| Concern | Whole-project audit | Revision docs |
|---|---|---|
| Security (OWASP Top 10) | `audit-security` | [docs/owasp/](docs/owasp/) |
| Testing (pyramid, test types, periodic disciplines) | `audit-test` | [docs/test-strategy/](docs/test-strategy/) |
| Performance (targets, code-level antipatterns, diagnosis, Core Web Vitals) | `audit-performance` | [docs/performance/](docs/performance/) |

## Caveat

Copilot has no equivalent of Claude's on-demand, description-triggered skill invocation — `copilot-instructions.md` is always loaded in full, and `*.instructions.md` files apply per their `applyTo` glob (currently `**` for all, i.e. always-on too). A skill written for occasional/manual Claude invocation may need trimming before it's suitable to keep permanently in Copilot's context. Review generated output per skill before relying on it, especially for large skills.

The generator copies only each skill's `SKILL.md` body. Supporting files next to it — e.g. `tdd/test-types.md`, `to-spec/performance-targets.md` — are **not** shipped to Copilot. Skills that point at them are written to stand on their own, but the deeper vocabulary is Claude-only.

## Skill format

Each `skills/<name>/SKILL.md` starts with frontmatter:

```markdown
---
name: skill-name
description: One line, used for Claude's auto-trigger matching and as the Copilot index blurb.
---

Skill body...
```

Keep skill content project-independent — no repo-specific names, paths, or examples. Project-specific skills stay in that project's own `.claude/skills/`, not here.

# Test Tools

Maps the tool-agnostic test vocabulary in `tdd/test-types.md` to the concrete tools and commands **this repo** actually uses. The engineering skills read this to name real tools and commands instead of generic advice. Where a row is blank, they treat that test type as "not set up here" and stay agnostic.

Fill in the tools your repo uses; delete rows that don't apply. **If this file doesn't exist, skills stay fully tool-agnostic** (they advise by test *type*, never by a tool they can't confirm you use).

## How the skills use this

- **`/tdd`** — when writing a test at a chosen level (unit/integration/…), use the runner and command listed here.
- **`/implement`** — when the work needs a contract / integration / e2e test, reach for the tool named here rather than inventing one.
- **`/audit-test`** — when inventorying the suite, check whether these tools are actually present and current. A blank **Mutation** row means no test-effectiveness measurement; a blank **Load** row on a performance-critical service is a finding.
- **`/to-spec`** — when recording *Test levels* decisions, name the concrete tool so the ticket isn't ambiguous.

The tool-agnostic definitions these map onto live in `tdd/test-types.md` — this file is the project-local half, that file is the vocabulary.

## Mapping

Fill one table per stack. Single-stack repos need only the first; a repo that spans (say) a backend and a mobile app repeats the table under a second `### <stack>` heading, so each skill picks the tool for the file it's touching.

### <stack — e.g. backend>

| Test type / discipline | Tool | Command | Notes |
|---|---|---|---|
| Unit | | | |
| Integration | | | |
| Component | | | |
| Contract (consumer / provider) | | | |
| End-to-end | | | |
| Snapshot | | | |
| Mutation | | | |
| Load / performance | | | |
| Coverage report | | | |

<!-- Example — replace with your own tools, delete rows that don't apply.
     Examples from different ecosystems (not tied to a single stack):

  ### .NET backend
  | Unit                  | xUnit                          | dotnet test                              |                         |
  | Integration           | WebApplicationFactory + test DB| dotnet test --filter Category=Integration|                         |
  | Contract              | Pact                           | (consumer publish + provider verify)     | consumer + provider     |
  | Mutation              | Stryker.NET                    | dotnet stryker                           | mutation score          |
  | Load / performance    | k6                             | k6 run load.js                           |                         |

  ### React Native / Expo
  | Unit + snapshot       | Jest                           | npm test                                 | stub timestamps/ids     |
  | Component             | React Native Testing Library   | npm test                                 |                         |
  | End-to-end            | Detox (or Maestro)             | detox test                               | mobile                  |
-->

## Keeping it honest

A tool listed here is a claim the repo actually uses it. When a skill can't find the tool where this file says it is (no config, no CI step, no dependency), that's a gap to surface — not a reason to invent a different tool. Update this file when the repo's tooling changes; a stale mapping sends skills to a tool that isn't there.

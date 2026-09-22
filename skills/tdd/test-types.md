# Test Types

Vocabulary for classifying tests by what they exercise and how far they reach — not how to write one (see [tests.md](tests.md)) or when to mock (see [mocking.md](mocking.md)). Two complementary frames, then the types themselves, then the periodic/whole-project ones that don't fit a red-green loop.

## Two frames

**The pyramid** (Fowler) says how the types should be *proportioned*: many fast, narrow tests at the base: fewer, broader ones as you go up. It doesn't define a boundary for any layer — teams disagree on where "unit" stops and "integration" starts.

**Sizes** (Google: Small / Medium / Large) fix that by rule instead of name: a **Small** test can't touch the network or filesystem and runs single-threaded; **Medium** may reach `localhost` and use the filesystem/DB, multi-threaded; **Large** has free network access to real external systems. Runtime budgets scale with size (Small tightest, Large loosest). Use sizes when a team argues about what "unit test" means — the rule ends the argument the name can't.

The two don't compete: the pyramid says how many of each, sizes say what each is allowed to touch.

## The types

| Type | What it exercises | Seam | When it's the right call |
|---|---|---|---|
| **Unit** | One component/class/method, dependencies isolated | The narrowest public interface around the unit | Every new piece of logic — the base of the pyramid, the fast-feedback default |
| **Integration** | Two or more real components together, infrastructure included (DB, filesystem, network) | A boundary between owned modules, or between a module and real infrastructure | Verifying a contract you can't verify by isolating either side — repository↔DB, controller↔service through the real wiring |
| **Component** | One service/deployable unit in isolation, its own dependencies doubled | The service's own public boundary | Confirming a service is internally correct without standing up the whole system |
| **Contract (consumer-driven)** | The agreement between a consumer and a provider it doesn't control | The wire format/API between two independently-deployed services | Two sides evolve independently and a full integration/e2e test between them would be slow or flaky; the contract also serves as living documentation of the agreement |
| **Subcutaneous** | End-to-end business logic, entered just below the UI | The layer directly under the UI (e.g. the API/controller a UI would call) | Wanting e2e-level confidence in a user-facing flow without paying for UI automation's fragility |
| **End-to-end / broad-stack** | The system as a whole, as close to production as practical | None — enters through the real front door (UI, public API) | Proving a critical flow works through every layer together — kept few, because it's the slowest and most brittle layer |
| **Acceptance / BDD** | A user story's acceptance criteria, phrased in domain language (Given-When-Then) | Whatever seam the criteria naturally exercise | The business side needs to read and agree with what's being verified, not just the engineering side |
| **Snapshot** | A component's rendered output against a stored reference | The component's render boundary | Catching unintended UI/output drift cheaply — never a substitute for a behavioral unit test, and unreliable if non-deterministic values (timestamps, ids) aren't stubbed |

Prefer the **highest seam that still gives you the confidence you need** — this is the same rule [SKILL.md](SKILL.md) gives for choosing seams generally, and it's why subcutaneous and component tests exist: they buy broad, integration-like confidence without an e2e test's cost.

## Smoke, sanity, regression

Same tests, different intent and timing — don't confuse the three:

| | Scope | Runs when | Answers |
|---|---|---|---|
| **Smoke** | Broad but shallow — the critical paths only | After every build/deploy | Is this build even worth testing further? |
| **Sanity** | Narrow — one recent change | After a small fix | Did that specific change work? |
| **Regression** | Deep — the system generally | After a meaningful change/refactor | Did this break something that already worked? |

## Whole-project disciplines (not a red-green loop)

These don't get written test-first inside a TDD cycle — they run periodically, against the whole test suite or the whole system, and belong to a project-wide review rather than a single red-green cycle.

**Mutation testing** answers a different question than coverage does. Coverage says a line *ran*; it says nothing about whether a test would notice that line breaking. Fowler's point (see [SKILL.md](SKILL.md)) is that coverage is a tool for finding untested code, not a quality signal — mutation testing is the check that actually measures test *effectiveness*: it seeds small deliberate faults ("mutants") into the code and checks whether the suite catches each one. A mutant the suite doesn't catch means that line is covered but not actually verified.

**Performance testing** has four distinct shapes, easy to conflate:

| Type | Question it answers |
|---|---|
| **Load** | How does the system behave under expected normal-to-peak traffic? |
| **Stress** | Where does it break under load beyond capacity, and how does it recover? |
| **Soak / endurance** | Does sustained high load surface a leak or degradation over time? |
| **Spike** | How does it react to a sudden, short burst (a special case of stress)? |

## Test doubles

Five distinct roles, often flattened into "mock" — worth naming precisely because they imply different assertions:

| Double | Role |
|---|---|
| **Dummy** | Fills a parameter slot; never actually used |
| **Fake** | A working, simplified stand-in (e.g. an in-memory store instead of a real DB) |
| **Stub** | Returns pre-programmed answers; no behavior verification |
| **Spy** | A stub that also records how it was called (count, arguments) |
| **Mock** | Pre-programmed with expected calls; the test fails if they don't happen as expected |

This splits into two verification styles: **state verification** (dummy/fake/stub/spy — assert on the result) and **behavior verification** (mock — assert the right call happened). Prefer state verification by default; it survives refactors that behavior verification doesn't, because it doesn't care *how* the result was reached.

This is also the fault line between two TDD traditions. **Classical (Detroit)** style uses real objects wherever practical and reaches for a double only when a real one is impractical (network, third-party, non-determinism) — see [mocking.md](mocking.md)'s "mock at system boundaries only". **Mockist (London)** style doubles every collaborator with interesting behavior and designs outside-in from the mocks. This skill defaults to Classical — it's what makes tests survive refactors (see the anti-patterns in [SKILL.md](SKILL.md)) — but naming the alternative matters when you inherit a codebase written the other way.

## Project-local tools

Everything above is tool-agnostic — it names *kinds* of test, never a runner or library. A repo maps these kinds onto its actual tools and commands in `docs/agents/test-tools.md` (scaffolded by `/setup-matt-pocock-skills` when a test skill is installed). **If that file exists, use its concrete tools and commands** — the runner for a given level, the repo's contract-test tool, its mutation and load tools. If it doesn't, stay agnostic: advise by test type, and prefer what the environment already reveals (test scripts in `package.json`/project files, existing test folders) over guessing a tool. A tool the mapping names but the repo can't be shown to use is a gap to surface, not a fact to trust.

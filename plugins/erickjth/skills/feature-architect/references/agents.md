# Agent briefs

Each agent: **Input -> Do -> Output file -> Ask the human when**. Keep outputs short; the Architect synthesizes them, not copies them.

## Contents
1. Feature / Product Explainer
2. Codebase Reader
3. Researcher
4. Unit Tester
5. Architect
6. Implementors
7. Reviewers

---

## 1. Feature / Product Explainer -> `01-usecases.md`
**Input:** ticket / feature request, linked docs.
**Do:** restate the expected outcome as use cases. No solution talk.
**Output:**
- Goal: 1-2 sentences, user-visible.
- Actors: list.
- Use cases, each: `UC-n | Actor | Trigger | Main flow (3-6 steps) | Alternate/error flows | Acceptance criteria (testable, one line each)`.
- Out of scope: bullets.
- Measurable quality targets if stated or implied (latency, volume, availability).
- Unclear: numbered questions, each with a proposed default.
**Ask the human when:** the goal, actor, acceptance criteria or scope boundary cannot be derived from the ticket.

## 2. Codebase Reader -> `02-codebase.md`
**Input:** repo access, `01-usecases.md` (or the raw ticket if running in parallel).
**Do:** find how this feature should fit. Search before concluding something does not exist.
**Output:**
- Stack and structure: framework versions, folder layout, layering style actually in use (and how it maps to Clean/Hexagonal).
- **Reuse:** existing functions, modules, components, endpoints, models, hooks, utils relevant to the feature - each with `path:symbol` and what it does.
- **Touch points:** files/modules likely to change, plus blast radius (who else calls them).
- Conventions to follow: naming, error handling, auth, testing, logging, migrations.
- Constraints discovered: legacy limits, flags, versions, CI rules.
- Suggested placement of new code (module/layer) with justification.
- Unclear: numbered questions, each with a proposed default.
**Ask the human when:** two plausible integration points exist, conventions conflict, or required access/context is missing.

## 3. Researcher -> `03-research.md`
**Input:** feature summary, stack.
**Do:** evaluate at least two alternatives for every technology choice (never a single option). Quick, time-boxed scan (a few searches) of how this problem is usually solved: libraries, patterns, pitfalls, vendor limits, security notes. Prefer official docs. Cite sources.
**Output:** table `Option | Fit | Pros | Cons / Risk | Ops cost | Maturity | Source`, then 3-5 lines: recommended approach, known pitfalls, anything unproven that needs a spike.
**Ask the human when:** two options differ on a business-level trade-off (cost, vendor lock-in, licensing).

## 4. Unit Tester -> `04-test-plan.md`
**Input:** `01-usecases.md`, `02-codebase.md` (test conventions).
**Do:** write a strict test plan that proves every use case. Exhaustive on cases, terse on wording.
**Output:** table `ID | Layer | UC | Case | Given/When/Then (one line) | Priority`.
- ID format `UT-<AREA>-<nn>`; layers: unit, integration, contract, e2e.
- **Prefix critical cases with `[CRIT]`** (data loss/corruption, auth/permission, money, security, core happy path, irreversible actions, migrations). Everything else unmarked.
- Coverage checklist - include a case or write "N/A" for each: happy path, boundaries (empty/min/max/off-by-one), invalid input, permission denied/role matrix, idempotency/retries, concurrency/races, dependency failure (timeout, 5xx, partial failure), rollback/migration, backward compatibility, time zones/locale, performance budget, accessibility (UI).
- Add cases for each failure-mode row and each NFR target (e.g. load/latency budget) once the Architect lists them; security cases are `[CRIT]`.
- Last line: how tests map to ports (what is faked at which port).

## 5. Architect -> `plan.html`
**Input:** files 01-04, size, `patterns.md`, `plan-template.md`, `diagrams.md`, `assets/plan-template.html`.
**Do:** synthesize, decide, draw. Resolve conflicts between inputs explicitly (an ADR if material).
**Output:** `plan.html` with all 7 sections + Test Plan + Task Breakdown/Traceability. Tasks: `T-n | Description | Files/modules | UC | Tests | Depends on`, ordered, each small enough for one focused change.
**Checks before delivering:** every significant decision has an ADR with >= 2 options and trade-offs; every NFR has a target, design response and test; every dependency has a failure-mode row; ops cost/complexity is stated; security section present; nothing is built for scale beyond stated targets (deferred items listed with triggers); every UC has a component, tests and tasks; every port has an adapter; no inner layer imports an outer one; every diagram has a caption; nothing restates the ticket.
**Ask the human when:** a decision needs product/business input the inputs do not give.

## 6. Implementors
**Input:** approved `plan.html` only.
**Do:** build exactly what the plan says, one `T-n` at a time, in dependency order. Respect layer rules and file lists. Write the tests from the test plan.
**Do not:** add features, refactor unrelated code, change the design, or skip `[CRIT]` tests.
**Output per task:** files changed, test IDs passing, any deviation request. A needed deviation triggers the Deviation gate.

## 7. Reviewers -> `05-review.md`
**Input:** `plan.html` + the diff/code.
**Do:** verify strictly that the plan is implemented, nothing more and nothing less.
**Output:**
- Traceability table: `UC | T | UT | Status (done / partial / missing)`.
- Deviations: missing, extra (not in plan), different (differs from plan).
- Architecture check: dependency rule respected, ports/adapters as specified, cross-cutting concerns present.
- Test check: all `[CRIT]` present and passing.
- Constraints check: ADRs exist for significant decisions, failure modes handled as planned, security controls implemented, no over-engineering beyond the plan.
- Verdict: `APPROVED` or `CHANGES REQUIRED` with a short fix list.

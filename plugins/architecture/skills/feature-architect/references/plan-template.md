# Plan template - sections and depth per size

S = small, M = medium, L = large. "-" = omit. Always: tables over prose, 2-sentence paragraphs max, one-line diagram captions.

## 1. Introduction and Goals
| Part | Content | S | M | L |
|---|---|---|---|---|
| Problem statement | Business/technical challenge, 2-3 lines | yes | yes | yes |
| Use cases | Diagram (actors -> UC) + table `UC / Actor / Flow / Acceptance` | <=3 | yes | yes |
| System scope | Two lists: In scope / Out of scope | yes | yes | yes |
| Quality attributes (NFRs) | Table `Attribute / Target / Design response / Test ID` - performance, scalability, security, availability, maintainability. Numbers, not adjectives; target = stated need + modest headroom | key 2-3 | all | all + SLOs |

## 2. Architecture Constraints and Assumptions
Two tables, `Constraint / Source / Impact`: technical (legacy, stack mandates, budget) and organizational (team size, skills, deadline). Add `Assumptions` as a short list; each assumption names who confirms it.
S: only what changes the design. M/L: full.

## 3. System Context and High-Level Design
- **Context diagram** (flowchart): users, external APIs, upstream/downstream systems around the feature. S/M/L.
- **Component map** (flowchart): frontend, API/gateway, core modules/services, datastores, queues, marking **new** vs **changed** vs **existing**.
- Decomposition choice (monolith / modular monolith / microservices / event-driven / CQRS / serverless) in one line + ADR link when non-default.

## 4. Detailed Views
| View | Content | S | M | L |
|---|---|---|---|---|
| Building blocks | Layer diagram (Clean) + ports/adapters table `Port / Direction / Adapter(s) / Fake` + module/folder tree with new/changed files | yes | yes | yes |
| Runtime | One sequence diagram per main UC; one for the main failure path; background job/event flow if any | 1 | 2-4 | all key flows + state diagram if stateful |
| Deployment | Cloud/containers/network topology, environments, config/flags, rollout | - (unchanged) | only if changed | yes |
| Data architecture | ER diagram, migration steps (expand -> migrate -> contract), backfill, caching layers + invalidation, retention | only if data changes | yes | yes + rollback plan |

## 5. Cross-Cutting Concerns
Table `Concern / Decision / Where implemented`:
- Security and auth: identity, RBAC/permission matrix (`Role x Action`), encryption in transit/at rest, input validation, secrets, PII.
- Logging and monitoring: structured logs, error tracking, metrics, alerts with thresholds, dashboards.
- Error handling and resilience: timeouts, retries (with backoff + idempotency keys), circuit breakers, fallbacks, error taxonomy.
- **Failure modes** table `Component/Dependency / Failure / Detection / Impact / Mitigation`: one row per external call, queue/async step, migration, and datastore. No row, no design.
- **Operational complexity and cost** table `Item / Estimate / Owner`: infra spend, new moving parts, on-call burden, deploy and rollback steps, runbook needs.
Security is never omitted, even for S. S: only concerns the feature touches (plus security + failure modes). M/L: everything.

## 6. Architecture Decision Records
One block per real decision (skip trivia):
```
ADR-n: <title>   Status: proposed | accepted
Context: 1-2 lines (drivers: NFRs, constraints)
Options: >= 2, each with + pros / - cons / ops cost
Decision: 1 line
Alternatives rejected: A (why not), B (why not)
Consequences: + gains / - costs (incl. operational burden)
Revisit when: trigger (e.g. "traffic > N rps")
```
S: 0-1. M: 1-3. L: all significant, incl. decomposition and data choices.

## 7. Risks, Technical Debt and Roadmap
- **Deferred scale** list: what is intentionally not built now, with the trigger to revisit.
- **Risks** table: `Risk / Likelihood / Impact / Mitigation / Owner` - SPOFs, scaling bottlenecks, unproven tech (spike needed?).
- **Phase plan** (gantt or table): MVP milestones with exit criteria -> post-launch refactors.
- **Tech-debt backlog**: items knowingly deferred, with trigger to revisit.
S: 2-3 risks, one phase. M: 2 phases. L: 3+ phases with rollout/rollback and exit criteria.

## Appendices (always)
- **A. Test Plan**: from `04-test-plan.md`, `[CRIT]` rows highlighted.
- **B. Tasks and Traceability**: table `T-n / Description / Files / UC / Tests / Depends on`, plus matrix `UC -> Components -> Tests -> Tasks`.
- **C. Open questions and changelog**: unresolved items; every post-approval change as a dated line.

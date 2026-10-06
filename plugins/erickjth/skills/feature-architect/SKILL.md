---
name: feature-architect
description: Design a concise, visual, technical architecture plan for a feature (small, medium or large) using a multi-agent workflow - codebase reader, researcher, product explainer, unit tester, architect, implementors, reviewers - and deliver it as an HTML artifact with Mermaid diagrams, use cases, ADRs, test plan and phased roadmap. Use this skill whenever the user wants to plan, design, architect, scope, or write a technical spec / RFC / design doc for a feature, ticket, epic or refactor, or says "plan this feature", "how should we build X", "architecture plan", "design before coding", or wants implementation split into plan, build, review - even if they don't say "architecture". Applies Clean Architecture, Hexagonal (ports and adapters) and system decomposition (monolith / modular monolith / microservices, event-driven, CQRS, serverless).
---

# Feature Architect

Turn a feature request into a **short, visual, technical plan**, then build and verify it strictly against that plan. The plan is the contract: implementors build only what it says, reviewers check only against it.

## Principles

- **Concise.** Tables, bullets and diagrams over prose. Max 2 sentences per paragraph. No filler, no restating the ticket.
- **Visual.** Every plan has diagrams (Mermaid) and use cases. A diagram needs a one-line caption, nothing more.
- **Traceable.** Every use case (`UC-n`) maps to components, tests (`UT-*`) and tasks (`T-n`). This is what makes strict implementation and review possible.
- **Grounded.** Reuse what the codebase already has before proposing anything new.
- **Human in the loop.** Never guess at ambiguity; batch questions and ask (see Gates).

## Constraints

These are non-negotiable at every size. They exist because plans fail in the gaps they close: unrecorded decisions, unexamined alternatives, happy-path-only designs, and unowned operational cost.

**MUST**
- **ADR every significant decision** (framework, datastore, pattern, build-vs-buy, decomposition shape), each with rejected alternatives.
- **Address non-functional requirements explicitly** - measurable targets in Section 1, each traced to a design choice and a test.
- **Evaluate trade-offs, not just benefits** - every option and ADR lists costs and downsides, not only gains.
- **Plan for failure modes** - a failure-mode table (what breaks, detection, impact, mitigation) for every external dependency and async step.
- **Consider operational complexity and cost** - who runs it, on-call burden, deploy/rollback, monitoring, infra spend. Put it in the plan, not an afterthought.
- **Review with stakeholders before finalizing** - Gate B is a stakeholder review, not a formality; record who approved.

**MUST NOT**
- **Over-engineer for hypothetical scale** - design for stated targets plus modest headroom; defer the rest to the roadmap with a trigger ("revisit when X > Y").
- **Choose technology without evaluating alternatives** - at least two options compared per technology decision.
- **Ignore operational costs** - a design with no run-cost or ops-burden line is incomplete.
- **Design without understanding requirements** - if use cases or NFRs are unclear, stop at Gate A; do not fill gaps with assumptions.
- **Skip security** - Section 5 security is mandatory even for size S (authn/authz, input validation, data sensitivity, secrets).

## Step 0 - Intake and sizing

Collect the ticket/request text, then pick a size. Size decides depth, not structure; the 7 sections always exist.

| Size | Signal | Plan depth |
|---|---|---|
| **S** | 1 module, no new data model, <= ~2 days | Sections 1, 3, 4 (runtime only), 5 (security + failure modes, never skipped), 7 short. 1-2 diagrams, <= 3 use cases, inline ADR only if a real choice exists |
| **M** | Several modules, schema change or new integration, <= ~2 weeks | All 7 sections. 4-6 diagrams, 1-3 ADRs, test plan by layer |
| **L** | New service/boundary, migration, multi-team, > 2 weeks | All 7 sections in full + deployment view, migration strategy, spikes for unproven tech, phased roadmap with exit criteria |

State the chosen size and why in one line. Switch size later if findings contradict it.

## Workflow

Work files live in `.feature-plan/<slug>/` (or `/home/claude/feature-plan/<slug>/`). Each agent writes one file; the next reads it. This keeps context small and the handoff auditable.

If a subagent tool exists, dispatch each wave in parallel; otherwise run the roles sequentially yourself. Full briefs, inputs and output contracts are in `references/agents.md` - read it before dispatching.

```
Wave 1 (parallel)   Explainer -> 01-usecases.md
                    Codebase reader -> 02-codebase.md
                    Researcher -> 03-research.md
GATE A              Merge open questions, ask the human
Wave 2              Unit tester -> 04-test-plan.md
Wave 3              Architect -> plan.html  (+ plan.md summary)
GATE B              Human approves the plan
Wave 4              Implementors -> code, one task (T-n) at a time
Wave 5              Reviewers -> 05-review.md  (plan vs. code, verdict)
```

### Gates (human in the loop)

- **Gate A** - after Wave 1, merge every "unclear" item from the Explainer and Codebase reader into one numbered list (max 5, each with your proposed default). Use `ask_user_input_v0` for choices, plain text otherwise. Do not proceed on silent assumptions.
- **Gate B (stakeholder review)** - present the plan to the human and name the stakeholders who should review it (product, security, ops/on-call, affected teams). Wait for approval and record who approved in the changelog. Implementors start only after this.
- **Deviation gate** - if an implementor or reviewer finds the plan is wrong or incomplete, stop, amend the plan (add a changelog line), get approval, continue. Never silently improvise.

## Architecture rules the Architect applies

Read `references/patterns.md` for the decision tables. In short:

1. **Dependency Rule (Clean Architecture):** dependencies point inward. Domain/use cases know nothing about frameworks, UI, ORM or HTTP.
2. **Ports and Adapters (Hexagonal):** the domain exposes ports (interfaces); DB, queues, third-party APIs and UI are adapters. Name each port and adapter in the building-block view.
3. **Right-size the decomposition:** default to the simplest option that meets the quality attributes (usually a modular monolith). Choose microservices, event-driven, CQRS or serverless only when a stated quality attribute demands it, and record why in an ADR.
4. **Match the codebase.** If the repo has its own conventions, follow them and map them onto these patterns rather than imposing a new structure.

## Plan template (the 7 sections)

Build `plan.html` from `assets/plan-template.html`. Section content and per-size depth: `references/plan-template.md`. Diagram snippets: `references/diagrams.md`.

1. Introduction and Goals - problem, scope in/out, quality attributes / NFRs (measurable), use cases
2. Constraints and Assumptions - technical and organizational
3. System Context and High-Level Design - context diagram, component map
4. Detailed Views - building blocks, runtime sequences, deployment, data architecture
5. Cross-Cutting Concerns - security/auth, logging/monitoring, error handling/resilience, failure-mode table, operational cost and complexity
6. ADRs - one short record per real decision, with rejected alternatives
7. Risks, Technical Debt and Roadmap - risks table, phase plan (MVP -> post-launch), tech-debt backlog

Plus two appendices that make the plan executable: **Test Plan** (from the Unit Tester) and **Task Breakdown + Traceability** (`UC -> component -> tests -> tasks`).

## Delivering the plan

- Write one self-contained HTML file (inline CSS/JS, Mermaid in `<pre class="mermaid">`), light and dark aware.
- If the Artifact tool is available, publish it; otherwise save to `/mnt/user-data/outputs/` and call `present_files`.
- Reply in chat with: size chosen, 3-5 line summary, open risks, and the Gate B question. Do not repeat the plan.

## Implementation and review

- **Implementors:** read `plan.html` only. Do tasks in order, one `T-n` at a time, each with its tests from the test plan written first or alongside. Touch nothing outside the task's listed files/modules. Report each task as done with the test IDs passing.
- **Reviewers:** check the code against the plan, not personal taste. Output a traceability table (`UC / T / UT -> status`), list of deviations (missing, extra, different), and a verdict: `APPROVED` or `CHANGES REQUIRED`. Any failing `[CRIT]` test or missing use case = `CHANGES REQUIRED`.

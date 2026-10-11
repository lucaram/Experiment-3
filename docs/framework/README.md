# AI quality framework

Status: **Proposed**. Owner: AI QE Lead.

## Purpose

AI can now write requirements-based test plans, tests and code faster than people can review them. This framework lets AI do that volume of work while people keep control of the decisions that matter: what "correct" means, and what is safe to release.

## Principles

1. **AI does the work; people approve what defines "correct".** Requirements, business rules, contracts, test plans and releases are approved by a named role. Everything else can be automated.
2. **Expected behaviour comes only from approved context.** Stories, business rules, architecture decisions and API contracts are the source of truth, never the application code. When the code and the context disagree, that is a possible bug, not a reason to change the test.
3. **Every gate is enforced by a check, not an instruction.** If a rule matters, CI or branch protection enforces it. An instruction to an AI agent is a guide, not a control.
4. **Everything is versioned and traceable.** Context, plans, tests, agent instructions and this framework live in git. Every test traces back to the requirement it proves, and every approval is recorded.
5. **Gates start strict and loosen with evidence.** Every gate begins with full human review and is relaxed only when the data shows reviewers rarely change what they approve.

## The loop

| Stage | What happens | Details |
|---|---|---|
| 1. Versioned context | Requirements, business rules, architecture, API contracts and agent instructions are pulled from their sources and pinned to a version. | [stage-1-context.md](stage-1-context.md) |
| 2. AI generates | AI writes test plans, tests, fixes for broken tests and application code from the context. | [stage-2-generate.md](stage-2-generate.md) |
| 3. Evaluate in CI | Deterministic checks, traceability, mutation testing and (for AI features) probabilistic checks decide pass or fail. | [stage-3-evaluate.md](stage-3-evaluate.md) |
| 4. Release gate | Each release is promoted, held or sent to human review based on Stage 3. | [stage-4-release.md](stage-4-release.md) |
| 5. Observe in production | Incidents, defects and AI-feature quality (drift, cost, latency) are monitored. | [stage-5-observe.md](stage-5-observe.md) |
| 6. Feed back | Confirmed failures become clarified requirements, regression tests and better agent instructions, and flow back into Stage 1. | [stage-6-feedback.md](stage-6-feedback.md) |

Each stage file has the same shape: **What** happens, **How** it is done, the **Human gate** (who approves what), and its **Status** (Proposed, Approved, Built).

## Roles

| Role | Decides |
|---|---|
| Product Owner | Requirements, business rules, test plans, and accepting business risk on releases |
| Solution Architect | Architecture decisions (ADRs, architecture model) |
| Software Engineer | API contracts, application code |
| Tech Lead | Releases, technical risk, AI-feature quality thresholds |
| QE Engineer | Test code and fixes to tests |
| AI QE Lead | The framework itself, agent instructions and skills, quality thresholds |

Each role is mapped to named people, and to the files they approve, in `roles.json`.

## Gate types

- **Always:** every item needs a person's approval.
- **On exception:** checks decide automatically, and a person acts only on what they flag.

## Tuning rule

Track how often people change what each gate approves, and move each gate one step at a time:

1. **Full review:** a person approves every item. Every gate starts here.
2. **Sampling:** a person reviews a random share (e.g. 1 in 10). Move here after a few months of "approved unchanged".
3. **Automatic approval:** only after the samples also stay clean for a long time, and only where a missed mistake is cheap to fix.

If changes become frequent at any step, move the gate back one step.

## How this framework changes

The framework is built and changed through pull requests, like any other code:

1. **Proposal PR:** a stage file (or a change to one) plus its entries in `docs/decisions.md`. Approved by the roles the change affects. This agrees *what* the rule is.
2. **Implementation PR:** the agents, skills, scripts and CI that put the approved proposal into practice. Approved by the AI QE Lead and a QE Engineer. This checks the *how* matches the proposal.

Only the roles a change affects are asked to approve it. Until the automated approval check exists, approvals are recorded as PR reviews.

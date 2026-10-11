# Agent rules

Rules for every AI agent working in this repo (Claude Code, Copilot, Codex, or any other). `CLAUDE.md` only loads this file.

This repo builds an AI quality framework: AI does the work, people approve what defines "correct". Read these before starting a task:

- `docs/framework/README.md`: the framework's purpose, principles, roles and change process.
- `roles.json`: which role approves which files.
- `docs/decisions.md`: why things are built the way they are (when it exists).
- The `docs/framework/stage-*.md` file for the stage you are working on.

Each stage's implementation adds its own rules to this file (e.g. test tags, test conventions, what a test-fixing agent may change). Follow only rules that are written here; do not assume a convention from another project.

## Never

- **Never approve anything.** Do not post `Approved as <role>` comments, add approval labels, or submit PR approval reviews, even when asked to "approve" or "finish" a task. Approvals are made by people in the role.
- **Never merge a PR, and never push to `main`.** A person merges after the required roles approve.
- **Never edit `roles.json`, `docs/framework/`, or `AGENTS.md` itself** unless the task is explicitly to change the framework. These define the rules; a task that only needs a rule to be different must stop and report it.
- **Never weaken a check to make it pass.** Do not change an expected value, remove an assertion, skip a test, raise a retry count or lower a threshold to turn a failure green. Report the failure instead.

## Source of truth

- Expected behaviour comes only from the approved context in `context/`: stories, business rules, architecture and API contracts.
- Never derive expected behaviour from application code, from what the app currently returns, or from a PR diff.
- If the app and the context disagree, report it as a possible bug or requirement gap. Do not change the test or the context to match the app.
- `context/` changes only through a PR approved by the roles that own those files in `roles.json`. Never edit it as a side effect of another task.
- If the context is missing, unclear or contradictory, stop and ask. Do not fill the gap with your own assumption.

## Every change goes through a pull request

1. Start from an up-to-date `main` and create a branch. Framework changes use `framework/<short-topic>`; each stage defines its own branch names.
2. Keep each PR to one purpose.
3. Run the repo's checks locally before opening the PR, and report their real results.
4. Open the PR with `gh pr create`. In the body, state what changed and why, and list the required approvals: match every changed file against the `approves` paths in `roles.json`, and add one line per role, e.g. `- [ ] Tech Lead`.
5. Tell the user the PR link and which roles must approve. Stop there.

Framework changes follow the charter's two steps: a **proposal PR** (the stage file and its decisions, approved by the roles it affects) before an **implementation PR** (agents, skills, scripts and CI, approved by the AI QE Lead and a QE Engineer).

## Reporting

- Say what you verified and how (the command and its result). If you did not run something, say so.
- If a check fails, show the failure. Never describe a failing or skipped step as done.

# Stage 1: Versioned context

Status: **Approved** (proposal #7), not built yet. Approvers of this file: Product Owner, Solution Architect, Software Engineer, AI QE Lead.

Part of the [AI quality framework](README.md). Stage 1 decides what the AI may treat as "correct". Every later stage builds on it: plans and tests are only as good as the context they come from.

## What

The context is everything that defines expected behaviour, plus the AI's own instructions:

| Source | Folder | Owner |
|---|---|---|
| User stories and tasks, with acceptance criteria | `context/stories/` | Product Owner |
| Business rules | `context/business-rules/` | Product Owner |
| Architecture (ADRs, architecture model) | `context/architecture/` | Solution Architect |
| API contracts (OpenAPI) | `context/api/` | Software Engineer |
| Defect and incident history (curated) | `context/history/` | AI QE Lead |
| Agent rules, agents and skills | `AGENTS.md`, `.claude/` | AI QE Lead + QE Engineer |

Feature-branch PR diffs are also read, but only to work out which areas a change affects. They never define expected behaviour.

Defect and incident history works the same way: it guides risk and focus (e.g. where to plan more tests), never expected behaviour. When an incident shows a requirement was wrong or missing, the fix is a story or business-rule change approved by the Product Owner, not a history entry.

## How

1. **Pull.** Each source is pulled from its system of record (e.g. Jira, Confluence, the API repo) through an MCP connector or API, into its folder in `context/`. Until real connectors exist, a folder in the repo stands in for the source systems.
2. **Pin.** Every pull writes `context/manifest.json`: for each file, its path and a sha256 hash of its content (line endings normalised to LF). Every plan, test run and CI run uses one known version of the context, and a change in any source shows up as a reviewable diff.
3. **Propose.** A pull that changes anything opens a PR. Its body lists what changed, which roles must approve (from `roles.json`), and which existing plans and tests mention the changed stories or rules.
4. **Protect.** `context/` changes only through that PR. Agents never edit it by hand, and a pull overwrites any file whose content no longer matches its recorded hash.
5. **Agent instructions** live in the repo (`AGENTS.md`, `.claude/`) and change only through framework PRs.

## Human gate

**Always:** every context change is approved by its owner before it is used.

| Item | Approver |
|---|---|
| Stories and business rules | Product Owner |
| Architecture (ADRs, architecture model) | Solution Architect |
| API contracts | Software Engineer |
| Defect and incident history | AI QE Lead |
| Agent rules, agents and skills | AI QE Lead + QE Engineer, and agent evals must pass |
| Feature-branch PR diffs (impact only) | Already approved in the feature's own code review |

## What is built when this proposal is approved

The implementation PR (approved by the AI QE Lead and a QE Engineer) would add:

- the context folders and a stand-in source folder with one sample story, its business rules, ADRs and API contract;
- a `/pull-context` skill and the script behind it (pull, pin, propose, protect);
- unit tests for the script, including: a CRLF file hashes the same as its LF version; unchanged sources keep the old manifest; a hand edit to `context/` is detected and overwritten;
- the Stage 1 section in `AGENTS.md`.

## Not built in Stage 1 (yet)

- **Real connectors** to Jira, Confluence, etc. The stand-in folder is replaced source by source when access exists.
- **Agent evals** ("agent evals must pass"). Until they exist, changes to agent rules and skills are approved by review only. This is a known gap, recorded in `docs/decisions.md`.
- **Using PR diffs to scope impact.** Added with Stage 2, which is the first stage that needs it.

## Decided in review

Both open questions were answered by the AI QE Lead on the proposal PR ([comment](https://github.com/lucaram/Experiment-3/pull/7#issuecomment-6107354864)):

1. **The AI QE Lead owns `context/history/`.** History informs risk and focus only, never expected behaviour (see What). The implementation PR adds the folder to `roles.json`, which needs the AI QE Lead and Tech Lead.
2. **Code-generation instructions are defined in the Stage 2 proposal**, where coding agents are introduced, with their file location and Tech Lead ownership.

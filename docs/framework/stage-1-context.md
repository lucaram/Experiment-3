# Stage 1: Versioned context

Status: **Approved** (proposal #7, amended #9), not built yet. Approvers of this file: Product Owner, Solution Architect, Software Engineer, AI QE Lead.

Part of the [AI quality framework](README.md). Stage 1 decides what the AI may treat as "correct". Every later stage builds on it: plans and tests are only as good as the context they come from.

## What

The context is everything that defines expected behaviour, plus the AI's own instructions. Each item stays in its own system of record; the repo records **which version** of it was used.

| Source | Example systems (to be decided per company) | Version recorded | Owner |
|---|---|---|---|
| User stories and tasks, with acceptance criteria | Jira, Azure DevOps | ADO: revision number. Jira: `updated` timestamp + content fingerprint | Product Owner |
| Business rules | Jira, Confluence | Confluence: page version. Jira: `updated` timestamp + fingerprint | Product Owner |
| Architecture (ADRs, architecture model) | Confluence, Structurizr in git | Confluence: page version. Git: commit | Solution Architect |
| API contracts (OpenAPI) | git | Git: commit | Software Engineer |
| Defect and incident history (curated) | Jira, ADO; the curated list lives in `context/history/` | as for stories | AI QE Lead |
| Agent rules, agents and skills | this repo (`AGENTS.md`, `.claude/`) | Git: commit | AI QE Lead + QE Engineer |

"Content fingerprint" is a sha256 hash of the item's text with line endings normalised to LF: it shows whether the text changed, even where the tool keeps no version number.

Feature-branch PR diffs are also read, but only to work out which areas a change affects. They never define expected behaviour.

Defect and incident history works the same way: it guides risk and focus (e.g. where to plan more tests), never expected behaviour. When an incident shows a requirement was wrong or missing, the fix is a story or business-rule change approved by the Product Owner, not a history entry.

## How

1. **Read live.** Agents read each item from its system of record through an MCP connector or API, at the moment they need it. Context is not copied into the repo by default.
2. **Pin the version.** Every run that uses context (planning, test generation, CI) records in `context/lock.json`, for each item: source system, ID, the version the tool offers (see What), and whether that version was approved. Like a package lock file, it makes every plan and test traceable to the exact requirement versions they came from.
3. **Copy only when the tool cannot return an old version.** For tools like Jira, where old text can only be rebuilt from history, the exact text used is saved as a snapshot next to the lock entry. Company content is copied only into a private repo or store, never a public one.
4. **Approve where the owner works.** Owners approve in the source system (e.g. a story's status set to Approved) or, for sources kept in git, through a PR. The lock records which approved version was used, and a check refuses versions that were not approved.
5. **Detect stale context.** When a source item changes after the version in the lock, the plans and tests built on it are flagged for re-planning. Updating the lock opens a PR that lists what changed, the roles that must approve it (from `roles.json`), and the affected plans and tests.
6. **Stand-in until connectors exist.** A `sources/` folder in this repo plays the source systems. Its files are versioned by git commit, so the stand-in works like a git-based source.
7. **Agent instructions** live in the repo (`AGENTS.md`, `.claude/`) and change only through framework PRs.

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

- the `sources/` stand-in with one sample story, its business rules, ADRs and API contract;
- a source adapter for the stand-in (reads an item and returns its text and git commit), so real adapters can be added later without changing the rest;
- a `/pull-context` skill and the script behind it: read the items a story needs, write `context/lock.json`, detect items that changed since the lock, and open the lock-update PR with required roles and impact;
- unit tests for the script, including: a CRLF text has the same fingerprint as its LF version; an unchanged source leaves the lock unchanged; a changed source is detected and its dependent plans and tests are listed;
- the Stage 1 section in `AGENTS.md`.

## Not built in Stage 1 (yet)

- **Real adapters** for Jira, Azure DevOps, Confluence, etc. The company's source systems are chosen first; then each source switches from `sources/` to its real adapter.
- **Snapshots** for tools that cannot return old versions. Not needed for the stand-in, which is versioned by git; added with the first Jira-like adapter, in a private store.
- **The approval check on the lock** (refusing unapproved versions). Built with Stage 3's CI checks.
- **Agent evals** ("agent evals must pass"). Until they exist, changes to agent rules and skills are approved by review only. This is a known gap, recorded in `docs/decisions.md`.
- **Using PR diffs to scope impact.** Added with Stage 2, which is the first stage that needs it.

## Decided in review

Both open questions were answered by the AI QE Lead on the proposal PR ([comment](https://github.com/lucaram/Experiment-3/pull/7#issuecomment-6107354864)):

1. **The AI QE Lead owns `context/history/`.** History informs risk and focus only, never expected behaviour (see What). The implementation PR adds the folder to `roles.json`, which needs the AI QE Lead and Tech Lead.
2. **Code-generation instructions are defined in the Stage 2 proposal**, where coding agents are introduced, with their file location and Tech Lead ownership.

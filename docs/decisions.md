# Decisions

Why this repo is built the way it is. Read this before changing the framework or building a stage.

Each entry says what was decided and why, with the PR that made it. Add new decisions at the end, numbered, through the same PR as the change they explain. A decision that replaces an earlier one says so, and the earlier one is marked **Superseded by #N** instead of being deleted.

The previous build of this framework (repo `lucaram/ai-qe-loop`, Stages 1–3) is a reference, not a source: its choices are reconsidered stage by stage here and recorded below when adopted.

## How the framework is built

1. **The framework is built one file at a time, each through a pull request.** People in the affected roles approve each step, so they share ownership of the rules instead of having them imposed, and every rule is version-controlled with its approval. (#1)
2. **The charter comes first** (`docs/framework/README.md`): purpose, principles, the loop, roles, gate types, the tuning rule and the change process. Every later file builds on it. It names no tools, so it stays valid if a tool changes; tools belong in the stage files. (#1)
3. **Two PRs per stage: proposal, then implementation.** The proposal (stage file plus decisions) is approved by the roles the stage affects; the implementation (agents, skills, scripts, CI) by the AI QE Lead and a QE Engineer. Non-technical approvers judge rules, not code. (#1)

## Repository

4. **Public GitHub repo** (`lucaram/Experiment-3`). Branch protection is free on public repos; on private repos it needs a paid plan. (#1)
5. **`main` is protected** (GitHub branch protection, set through the API, not stored in the repo):
   - every change needs a pull request;
   - 0 approvals required, because GitHub does not let the owner approve their own PR and there is only one account;
   - applies to admins too (`enforce_admins`);
   - linear history (squash or rebase merges only);
   - no force-pushes, no deletion of `main`;
   - no required status checks yet; Stage 3 adds them.

   To see the live settings: Settings → Branches, or `gh api repos/lucaram/Experiment-3/branches/main/protection`. Because the settings live only in GitHub, an admin could change them without a PR. Planned: a setup section in the README with the exact settings, and a check that compares them with GitHub's. (#5)
6. **Line endings are LF** for all text files (`.gitattributes`), so file hashes and diffs are the same on Windows and Linux. (initial commit)

## Roles and approvals

7. **Six roles:** Product Owner, Solution Architect, Software Engineer, Tech Lead, QE Engineer, AI QE Lead. The Developer role was merged into Software Engineer, which owns both API contracts and application code. (#1, #2)
8. **`roles.json` is the single source of who approves which files.** Each role lists its people (`users`) and the paths it approves (`approves`). When real people join, only the `users` lists change. A later approval check will read this file. (#2)
9. **A PR needs the roles whose paths it changes.** Each stage file is owned by the roles that stage affects (e.g. the Stage 4 release gate: AI QE Lead, Tech Lead, Product Owner), not by everyone. (#2)
10. **Changing `roles.json` needs two roles: AI QE Lead and Tech Lead.** The file decides who can approve everything else, so one person must not be able to change it alone. (#2)
11. **Paths are owned before the files exist** (`app/`, `tests/`, `scripts/`, CI, tool config), so every file has an owner from the day it arrives. (#2)
12. **Until the approval check exists, approvals are PR comments: `Approved as <role>`**, one per required role, before merging. This is recorded but not enforced. A PR comment is preferred over writing the approval in the squash commit message: GitHub shows who wrote a comment, while a commit message is written by whoever merges, so it cannot prove which person approved. Repeating the approval in the commit message as well is fine. (#3)
    - Known gaps (PRs #1–#4, before this rule was followed):
      - Approvals were recorded only in the squash commit message, not as PR comments.
      - #4 recorded only the AI QE Lead; the QE Engineer approval its `AGENTS.md` change needed is missing.
      - Nothing blocked these merges. The approval check (Stage 3) will close this gap.

## Context

13. **Context folders have clean names:** `context/stories/`, `context/business-rules/`, `context/architecture/`, `context/api/`. They replace `jira-requirement` and `budiness-rule` from the previous build. (#2)

## Agents

14. **Agent rules live in `AGENTS.md`**, the cross-tool standard, so the same rules apply whatever AI tool is used. `CLAUDE.md` contains only `@AGENTS.md`, which makes Claude Code load it. Neither file has `name`/`description` frontmatter: that format is for skills and subagents, which a tool chooses between; these files are always loaded. (#4)
15. **`AGENTS.md` holds only rules that apply now.** Each stage's implementation PR adds that stage's rules (e.g. test tags, what a test-fixing agent may change), so the file never describes something that does not exist yet. (#4)
16. **Agent rules are instructions, not controls, until checks back them.** Today only "never push to `main`" is enforced, by branch protection. An agent could still merge a PR or post an `Approved as` comment, because it uses the owner's GitHub login. Planned controls: the approval check (Stage 3) and a separate GitHub identity for agents with no merge rights. (#4)

## Stage 1: Versioned context (proposal)

17. **Context is a pinned snapshot.** Each source is pulled into its `context/` folder, and `context/manifest.json` records a sha256 per file. Every run uses a known version, and any change is a reviewable PR. (#7)
18. **Hashes are taken over LF-normalised content**, so a file hashes the same on Windows and Linux. (#7)
19. **`context/` changes only through a pull PR.** A pull also overwrites hand-edited files whose content no longer matches the manifest, so the snapshot cannot drift silently. (#7)
20. **A stand-in source folder replaces the real systems until connectors exist.** The pull, pin and approval logic is the same either way; each source switches to a real connector when access exists. (#7)
21. **A pull PR lists the impact:** the existing plans and tests that mention the changed stories or rules, so approvers see what a context change will affect. (#7)
22. **Agent evals are a known gap.** The Stage 1 gate says agent evals must pass before agent rules or skills change; until they exist, those changes are approved by review only. (#7)

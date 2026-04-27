# AGENTS.md

This repository contains reusable Codex skills maintained by Waleed Salama.

## Repository Intent

- This repo is published as the public GitHub repository `waleed-salama/skills`.
- It packages reusable skills in the Vercel/OpenAI-compatible skill layout.
- The current primary skills are:
  - `linear-flow`: a structured Linear workflow from idea intake through triage, planning, implementation, review, closeout, and cancellation.
  - `commit-flow`: a delivery workflow for checking repository diffs, updating documentation, committing, pushing, and optionally creating pull requests.
- The repo is the source of truth for these skills. During local development, install or symlink these skill folders into the active agent's skills directory so edits here are immediately visible to new chats.

## Repository Layout

Use this public package layout:

```text
skills/
├── AGENTS.md
├── README.md
├── LICENSE
├── .gitignore
└── skills/
    ├── commit-flow/
    │   ├── SKILL.md
    │   └── agents/
    │       └── openai.yaml
    └── linear-flow/
        ├── SKILL.md
        ├── agents/
        │   └── openai.yaml
        └── references/
            ├── assertion-path.md
            ├── cancellation-protocol.md
            ├── direct-planning-issue-path.md
            ├── direct-triaged-issue-path.md
            ├── discovery-path.md
            ├── implementation-path.md
            ├── planning-path.md
            └── triage-path.md
```

## Local Development Symlink Pattern

For local development, the working tree can be symlinked into Codex's global skills directory:

```text
~/.codex/skills/linear-flow
  -> <repo-root>/skills/linear-flow

~/.codex/skills/commit-flow
  -> <repo-root>/skills/commit-flow
```

Do not hardcode maintainer-specific filesystem paths in skill files. Public skill content should use skill names, relative references, or placeholders such as `<repo-root>` and `<project-root>`.

## Skill Packaging Standard

- Each skill must remain a self-contained folder under `skills/<skill-name>/`.
- Every skill requires `SKILL.md` with YAML frontmatter containing at least `name` and `description`.
- `agents/openai.yaml` is recommended and should stay aligned with the skill.
- Detailed path-specific workflow instructions should live in `references/` and be directly linked from `SKILL.md`.
- Avoid adding extra agent-facing documentation inside individual skill folders unless it is needed by the skill. Repo-level docs such as this file and `README.md` are fine.

## Validation

Validate skills with:

```bash
python3 "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" "skills/linear-flow"
python3 "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" "skills/commit-flow"
```

Validate through the global symlinks with:

```bash
python3 "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" "$HOME/.codex/skills/linear-flow"
python3 "$HOME/.codex/skills/.system/skill-creator/scripts/quick_validate.py" "$HOME/.codex/skills/commit-flow"
```

Run validation after every meaningful skill edit.

## Public Install

Install both skills globally for Codex with:

```bash
npx skills add waleed-salama/skills -g -a codex --skill linear-flow commit-flow
```

Install one skill globally for Codex with:

```bash
npx skills add waleed-salama/skills -g -a codex --skill linear-flow
npx skills add waleed-salama/skills -g -a codex --skill commit-flow
```

List available skills without installing:

```bash
npx skills add waleed-salama/skills --list
```

For local development, prefer symlinks over copying so edits in this repo are immediately available to Codex after a new chat starts or the skill is re-read.

## Linear Flow Context

`linear-flow` is the more complex skill and has been shaped through extensive daily use in TweetWizard.

The current status model is:

- `Suggested`: agent suggestions and assertion cases that need review or validation.
- `Draft`: accepted fuzzy ideas needing triage.
- `Triaged`: clear problem, value, and rough scope; waiting for the user to manually prioritize.
- `Todo`: active user-prioritized unstarted queue. Planning sub-state is tracked with labels, not separate statuses.
- `In Progress`: implementation has started.
- `Blocked`: implementation is blocked.
- `In Review`: implementation is ready for user testing and review.
- `Done`: accepted and complete.
- `Canceled`: intentionally closed without implementation.
- `Duplicate`: closed because another issue tracks the work.

Manual user-only status promotions:

- `Suggested -> Draft`
- `Triaged -> Todo`

Legacy statuses:

- `Prioritized`, `Planning`, and `Ready` are obsolete as statuses.
- They must not be used for active path selection or promotion unless the user explicitly asks for migration or cleanup.

Planning labels:

- The `Planning` label group contains:
  - `Claimed`: a planning agent has claimed a `Todo` issue.
  - `Ready`: the `Todo` issue has an approved implementation-ready plan.
- Normal issues should have at most one label from the `Planning` group.
- When replacing planning labels, preserve all other labels.

Queue model:

- Triage Path selects the first 7 `Draft` issues in manual Linear order.
- Planning Path selects the first `Todo` issue without any `Planning` group label.
- Planning Path must add `Claimed` immediately after the ordered GraphQL query returns the issue id. Do not read the issue, inspect the repo, analyze, send a progress update, or make any other tool call between selection and claim.
- Implementation Path selects the issue just planned in the same chat when the user asks to implement after planning. Only when there is no explicit or current-chat issue does it select the first `Todo` issue with the `Ready` label.
- Manual queue order in Linear is authoritative.
- Use the direct Linear GraphQL API only for ordered queue selection, with the API key loaded from `LINEAR_API_KEY` or a local OS secret store inside the API-call process. After queue selection, use actual Linear MCP/app tools for issue reads and writes. If the ordered query cannot be completed authoritatively, stop. Do not fall back to native Linear issue listing order.

Repository identifier cache:

- Projects using `linear-flow` should keep `planning/linear.md`.
- That file should cache team, project, status, and label identifiers, including the `Planning` label group id and `Claimed` / `Ready` label ids.
- Agents should read `planning/linear.md` first and avoid rediscovering authenticated user, teams, projects, statuses, or labels when the cache is complete.

Pre-implementation storage:

- Discovery, Triage, Direct Triaged Issue, Direct Planning Issue, and Planning paths keep planning in Linear issue descriptions.
- Do not use Linear comments for pre-implementation shaping.
- Use comments only during implementation or later, or when recording cancellation/duplicate reasons.

Output visibility rule:

- User-facing deliverables must be in `final`, not only progress commentary.
- Do not send a second trailing message like "waiting for approval" after a deliverable because it can hide the useful output in collapsed UI history.
- Template-based user-facing output should render as normal Markdown, not inside fenced code blocks unless the user explicitly asks for raw Markdown.

Emoji rule:

- Emojis are allowed only in approved template headings in `linear-flow`.
- Do not add random decorative emojis elsewhere.

Planning behavior:

- Planning depth is `Fast`, `Standard`, or `Deep`.
- Think deeply, but keep visible output concise.
- Ask only material questions whose answers would change implementation.
- For `Fast` issues with no material decisions, skip Planning mode and ask for approval directly.
- For `Standard` and `Deep`, ask questions in batches of 3 to 5.
- After Planning-mode questions are complete, the agent must ask the user to switch out of Planning mode before drafting the plan, and must explicitly warn the user not to use the built-in `Implement this plan` control.
- Planning finalization replaces `Claimed` with `Ready` and leaves the issue in `Todo`.
- If planning is abandoned before `Ready`, ask whether to release the claim. If approved, remove only `Claimed`.
- The old `Claimed Description Template` was removed. `Claimed` is only a lock label.
- The only detailed persisted planning template is the `Ready` description template after user approval.

Implementation behavior:

- Implementation starts from a `Ready`-labeled `Todo` issue. If the chat just finished planning an issue and the user asks to implement, use that issue instead of querying for the top `Ready` issue.
- The issue is not moved to `In Progress` until the user approves the implementation approval-gate message.
- After approval, move the issue to `In Progress` and remove `Ready`.
- If repository code changes are required, the chat must already be inside a dedicated worktree. The agent must not create a worktree. If not inside one, stop and ask the user to hand off the chat into a Codex UI worktree.
- Once inside a worktree and approved, create the issue branch immediately so the IDE shows the branch.
- Do not create temporary clones, copied repos, copied worktrees, backup directories, or side workspaces.
- Keep all implementation, validation, closeout, commit-flow, and PR preparation inside the active worktree.
- For worktree env-dependent commands, source env files from the parent repo instead of copying `.env` files into the worktree.
- Use approved dev server ports only: `3000` through `3005`. If all are taken, stop and ask the user.
- Never terminate a dev server this session did not start.

Implementation closeout:

- After user acceptance, ask whether any known non-blocking annoyance should become a separate `Suggested` follow-up issue.
- Remove any temporary implementation note from `AGENTS.md` before mergeability probe or commit-flow.
- Run the pre-commit mergeability probe before creating a real commit.
- Use `commit-flow` to commit, push, and create a GitHub PR targeting the recorded parent branch.
- Do not merge the issue branch back into the parent branch locally during this flow.
- Add the closeout comment.
- Move the Linear issue to `Done` manually. Do not wait for PR merge automation to move it.

Native Linear/GitHub linking:

- Prefer native PR linking through the PR description over manual PR URL attachments.
- Use only these magic-word forms:
  - `Part of WAL-123` for non-closing support links.
  - `Resolves WAL-123` for closing links.
- Even when `Resolves` is used, the agent still manually moves the Linear issue to `Done` after acceptance and closeout.

Admin issues:

- `Admin` is a normal kind label alongside `Feature`, `Improvement`, and `Bug`.
- Admin issues are primarily external configuration, dashboard, provider, policy, process, or operational work.
- Admin issues may require no code, some code, or mixed work.
- Implementation Path must not assume code changes for Admin issues.

Assertion issues:

- `Assertion` is an overlay label, not a normal kind label.
- Assertion issues live in `Suggested`.
- Assertion Path exists but has not been heavily used yet. Do not over-refactor it without real usage evidence.

## Commit Flow Context

`commit-flow` is used by `linear-flow` implementation closeout when repository changes exist.

Its role is to:

- inspect the repository diff
- audit relevant documentation requirements
- update stale docs before staging
- produce a commit message that reflects the full change set
- commit only, commit and push, or commit, push, and open a pull request depending on the requested mode

When editing `commit-flow`, preserve its role as a reusable delivery skill independent of Linear.

When `linear-flow` references `commit-flow`, prefer portable wording such as "use the `commit-flow` skill" instead of hardcoded local filesystem links.

## Current Handoff Status

- GitHub repository exists at `https://github.com/waleed-salama/skills`.
- The `skills` CLI can discover this package and currently lists `commit-flow` and `linear-flow`.
- The skill files have been scanned for maintainer-specific hardcoded local paths.
- Both skills validate with the `skill-creator` quick validator.
- Before future publishing updates, validate both skills, scan for hardcoded maintainer paths, commit the changes, and push `main`.

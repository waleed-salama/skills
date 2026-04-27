---
name: linear-flow
description: Route Linear work through a structured lifecycle from idea discovery to implementation. Use when Codex needs to add autonomous ideas to Linear, validate assertion issues from Suggested, create a new issue directly in Triaged from chat context, create a new issue directly into the planning workflow from chat context, triage Draft issues, deeply plan Todo issues without making code changes, or pull a Ready-labeled Todo issue into repository implementation with documentation, worktrees, testing, and commit-flow.
---

# Linear Flow

## Overview

Use this skill to manage Linear issues through a consistent backlog and delivery workflow. Keep pre-implementation work inside Linear, then switch to repository planning and implementation rules only after an issue is intentionally pulled into execution.

## Statuses

- `Suggested`: Pre-backlog intake for agent suggestions and assertion cases that still need review or validation before they become normal backlog work or are closed out.
- `Draft`: Accepted idea with fuzzy scope that needs shaping before product triage.
- `Triaged`: Clear problem, value, and rough scope; waiting for manual prioritization before entering the active queue.
- `Todo`: User-prioritized, unstarted issues queued for planning and implementation; planning sub-state is tracked with labels in the `Planning` label group.
- `In Progress`: Active implementation is underway in the repository.
- `Blocked`: Implementation has started but cannot proceed until a blocker is resolved.
- `In Review`: Implementation is complete enough for review, testing, and acceptance.
- `Done`: Implemented, reviewed, tested, and accepted as complete.
- `Canceled`: Intentionally closed without implementation.
- `Duplicate`: Closed because the work is already tracked by another issue.

## Shared Rules

- Use the issue description for all pre-implementation flows.
- Use comments only during implementation or later, or when recording a cancellation reason.
- Write real line breaks, not escaped newline sequences.
- Write valid Markdown.
- When showing the user content that is based on a skill template, render it as normal Markdown in the message body. Do not wrap that user-facing output in fenced code blocks.
- Fenced code blocks inside this skill are for defining templates and command examples only. They are not the required presentation format for user-facing deliverables.
- Prefer clean rewrites over endlessly appending stale notes.
- Think deeply, write briefly. User-facing outputs should contain the minimum text needed to make or confirm a decision without dropping material detail.
- Use emojis only through the approved `linear-flow` template-heading spec below. Do not add ad hoc or decorative emojis elsewhere in the skill outputs.
- Do not put emojis in issue titles, status names, labels, queue names, or normal narrative paragraphs unless a path template explicitly requires them.
- The approved template-heading emoji spec is:
  - `🧾` `Summary`
  - `📋` `Issue`
  - `❗` `Problem` or `Problem Statement`
  - `✨` `Value Proposition`
  - `📦` `Scope`
  - `🛠️` `Solution`, `Proposed Approach`, `Final Implementation Plan`, `Implementation Approach`, or `Execution Mode`
  - `🔧` `What Will Change`
  - `🧭` `Priority Rationale`
  - `📌` `Assumptions`
  - `🎯` `Goals`
  - `🚫` `Non-goals`
  - `🧩` `Current State`
  - `🗃️` `Data Model / Schema Impact`
  - `🔌` `Dependencies / Integrations`
  - `📈` `Analytics / Observability`
  - `🚦` `Rollout / Gating`
  - `🧪` `Validation` or `Validation / Testing`
  - `📝` `Repo Planning Required`
  - `❓` `Open Questions / Decisions`
  - `✅` `Acceptance Criteria` or `Approval Request`
  - `⛓️` `Constraints`
  - `🔎` `Evidence`
  - `➡️` `Proposed Next Step`
  - `🚀` `What Shipped`
  - `📚` `Documentation Impact` or `Docs Updated`
  - `🔁` `Notable Follow-up`
  - `🧷` `Commit`
- `Assertion` is a special-purpose overlay label, not a kind label.
- Use the `Assertion` label when the issue is a fear, edge case, pitfall, or expected-behavior claim that should be validated.
- `Assertion` issues live in `Suggested` and are handled only through the `Assertion Path`.
- By default, `Assertion` issues stay deferred in `Suggested` until the user intentionally invokes the `Assertion Path`.
- Do not put `Assertion` issues into `Draft` for normal triage.
- Apply exactly one kind label whenever the issue content is mature enough to classify it: `Feature`, `Improvement`, `Bug`, or `Admin`.
- Pure `Assertion` issues do not need one of the normal kind labels unless a separate normal work item is later created from a failed assertion.
- Treat those kind labels as follows:
  - `Feature`: net-new product capability or a meaningful new user workflow
  - `Improvement`: enhancement, optimization, polish, or refinement of existing behavior
  - `Bug`: incorrect, broken, regressed, or unreliable behavior
  - `Admin`: primarily administrative, configuration, dashboard, provider, operational, or process work that may involve little or no repository code
- `Admin` issues may still require code changes, but do not assume that they do.
- Use Linear priority as a decision-support signal, not as a replacement for the user's manual ordering.
- The following status changes are reserved for the user and must be done manually in the Linear UI:
  - `Suggested -> Draft`
  - `Triaged -> Todo`
- The user manually orders issues in Linear, and that manual order is the canonical queue order the agent must follow.
- The active planning and implementation queue uses the `Todo` status plus the `Planning` label group, not separate `Prioritized`, `Planning`, or `Ready` statuses.
- If the Linear workspace still contains `Prioritized`, `Planning`, or `Ready` statuses, treat them as legacy statuses for this workflow. Do not use them in active path selection or promotion unless the user explicitly asks for migration or cleanup work involving those statuses.
- The `Planning` label group has two workflow labels:
  - `Claimed`: an issue in `Todo` has been claimed by a planning agent and is being planned
  - `Ready`: an issue in `Todo` has an approved implementation-ready plan and can be selected by the Implementation Path
- When `planning/linear.md` contains the `Planning` label group id and child label ids, use those cached ids for queue filters and label updates instead of rediscovering them.
- If those label identifiers are missing or stale, fetch the team labels once, confirm the `Planning` group plus `Claimed` and `Ready` child labels, then update `planning/linear.md` when the repository uses ongoing Linear coordination.
- When creating or updating `planning/linear.md` for a project that uses this workflow, include a `Planning Labels` section with the label group name/id, `Claimed` label name/id, and `Ready` label name/id so future agents do not rediscover them.
- A normal issue should have at most one label from the `Planning` group at a time. If both `Claimed` and `Ready` are present unexpectedly, stop and ask the user before proceeding.
- When adding, removing, or replacing `Planning` group labels, preserve every non-Planning label already on the issue. Only change the `Claimed` and `Ready` labels unless the user explicitly asked for a broader label edit.
- Keep Discovery, Triage, and Planning inside Linear only.
- Start repository planning docs, architecture docs, and code changes only after a `Ready`-labeled `Todo` issue is approved for implementation and moved to `In Progress`.
- If the repository has mandatory planning rules for major changes, satisfy them during the Implementation Path before code changes begin.
- If the current repository contains `planning/linear.md`, read it first and treat it as the default source of truth for basic Linear identifiers and workflow notes.
- When `planning/linear.md` exists and is sufficient, do not call basic Linear discovery tools such as authenticated-user, team-list, project-list, or status-list just to rediscover that cached information.
- Only make those basic Linear discovery calls when `planning/linear.md` is missing, stale, incomplete, or clearly inconsistent with the current task.
- If `planning/linear.md` is missing or incomplete and the current repository uses ongoing Linear coordination, fill it in as soon as the needed identifiers are confirmed.
- Use the Linear plugin/skill for all normal issue reads and updates. Read first, then write.
- Direct Linear API usage is allowed only for ordered queue-selection GraphQL queries and setup smoke tests described in [references/linear-api-setup.md](references/linear-api-setup.md). After the ordered query returns issue identifiers, switch back to the Linear MCP/app for reading issue descriptions, labels, comments, status updates, and description writes.
- Use only Linear MCP/app tools that actually exist in the current environment. Typical tool names are `get_issue`, `update_issue`, `list_issues`, `list_issue_statuses`, `list_issue_labels`, `create_issue`, `list_comments`, and `create_comment`.
- Do not invent generic MCP tool names such as `research`. If the available Linear tool names are unclear, inspect the Linear plugin/skill/tool list first; if Linear MCP tools are unavailable or the needed tool is missing, stop and ask the user to connect or fix the Linear app instead of substituting direct API writes.
- When GitHub/Linear native integration is available, prefer native issue-to-PR linking over manual PR URL attachments.
- For implementation PRs, use the PR description as the authoritative place for native Linear linking.
- Use only `Part of {ISSUE-ID}` for non-closing links and `Resolves {ISSUE-ID}` for closing links.
- Do not treat manually pasting a PR URL into a Linear issue as the primary linking mechanism when native GitHub/Linear linking is available.
- Use the direct Linear GraphQL API only for queue-selection queries when the agent needs the actual manual order of issues in a status queue.
- When selecting issues from an ordered queue, do not rely on the standard Linear MCP issue listing order.
- If `planning/linear.md` contains the project id, use that cached id for queue-selection queries instead of rediscovering it.
- For direct Linear API setup, troubleshooting, or first-time user guidance, read [references/linear-api-setup.md](references/linear-api-setup.md).
- For queue selection, retrieve the Linear API key from `LINEAR_API_KEY` or a local OS secret store inside the same shell/process that performs the API call. Never ask the user to paste the key into chat.
- Never print, echo, log, commit, write to `planning/linear.md`, or show the Linear API key in chat.
- In a fresh thread with complete `planning/linear.md` identifiers, perform queue selection in one direct API call instead of doing a separate visible key-fetch step. Use an OS-appropriate one-shot command pattern such as:

```bash
sh -c 'k="${LINEAR_API_KEY:-$(security find-generic-password -a "$USER" -s codex-linear-api-key -w 2>/dev/null)}"; test -n "$k" || { echo "ERROR: Linear API key is not configured. See linear-flow references/linear-api-setup.md." >&2; exit 1; }; curl -sS https://api.linear.app/graphql -H "Authorization: $k" -H "Content-Type: application/json" --data-binary "$1"; unset k' sh '{"query":"query($projectId: ID!, $stateName: String!, $first: Int!) { issues(filter: { project: { id: { eq: $projectId } } state: { name: { eq: $stateName } } }, first: $first, sort: [{ manual: { order: Ascending } }]) { nodes { id identifier title sortOrder } } }","variables":{"projectId":"<project-id>","stateName":"Todo","first":1}}'
```

- The command above is the macOS Keychain pattern. On Linux, replace the key lookup with `secret-tool lookup service codex-linear-api-key account "$USER"` or another configured local secret command. On Windows PowerShell, use the SecretManagement pattern from [references/linear-api-setup.md](references/linear-api-setup.md).
- The canonical queue-selection query shape is:

```graphql
query($projectId: ID!, $stateName: String!, $first: Int!) {
  issues(
    filter: {
      project: { id: { eq: $projectId } }
      state: { name: { eq: $stateName } }
    }
    first: $first
    sort: [{ manual: { order: Ascending } }]
  ) {
    nodes {
      id
      identifier
      title
      sortOrder
    }
  }
}
```

- Queue-selection paths may add label filters to this base shape. Use these label filters exactly where the path calls for them:
  - first unclaimed planning issue in `Todo`: `labels: { every: { parent: { id: { neq: $planningGroupId } } } }`
  - first implementation-ready issue in `Todo`: `labels: { id: { eq: $readyLabelId } }`
- Use the `Planning` group parent id for excluding every issue that already has a planning workflow label.
- Use the specific `Ready` child label id for implementation selection. Do not filter implementation by the whole `Planning` group, because that would also match `Claimed` issues.
- For Planning Path selection, treat the ordered GraphQL result and adding `Claimed` as a critical claim sequence: once an issue id is returned, add `Claimed` immediately before any other Linear read, repository read, analysis, progress update, or user-facing message.
- Use a path-appropriate `first` value instead of overfetching:
  - `7` for Triage
  - `1` for Planning
  - `1` for Implementation
- When this manual sort query is used, treat the returned order as the true queue order without relying on standard Linear MCP listing order.
- If the manual-order queue-selection query cannot be completed reliably for any reason, stop immediately.
- Do not fall back to native Linear issue listing order, recently updated issues, recently created issues, or any other inferred ordering for queue-based paths.
- In that failure case, send one visible `final` message that explains the ordered queue could not be fetched authoritatively and asks the user to fix the Linear API key or queue-selection setup before continuing.
- Do not deviate from the path instructions in this skill by inventing side workflows, extra temporary repositories, copied workspaces, ad hoc clones, or hidden delivery steps outside the tracked workspace.
- If a path encounters a problem that the skill does not explicitly authorize a way to resolve, stop immediately and ask the user for further instructions instead of improvising a new workflow.
- If moving an issue to `Canceled` or `Duplicate`, read [references/cancellation-protocol.md](references/cancellation-protocol.md) first.

## Workflow Decision Tree

Choose exactly one path:

- `Discovery Path`: Use when the task is autonomous idea ingestion, competitor scouting, agent-led feature discovery, or codebase-driven improvement discovery. This path creates new issues in `Suggested` only and never makes code changes. Then read [references/discovery-path.md](references/discovery-path.md).
- `Assertion Path`: Use when the task is to validate the top `Suggested` issue labeled `Assertion`. This path handles assertion cases outside the normal backlog triage flow, may use repository work when assertion coverage or validation requires it, and either closes the assertion as validated or spins out normal follow-up work when the assertion fails. Then read [references/assertion-path.md](references/assertion-path.md).
- `Direct Triaged Issue Path`: Use when the user already discussed an issue in chat and wants Codex to create it directly in `Triaged` without first creating `Suggested` or `Draft`. This path drafts the same triaged structure used by the normal triage workflow, gets user approval, and then creates a brand-new Linear issue directly in `Triaged`. It never makes code changes. Then read [references/direct-triaged-issue-path.md](references/direct-triaged-issue-path.md).
- `Direct Planning Issue Path`: Use when the user already discussed an issue in chat and wants Codex to create it directly in the planning workflow without first taking it from the `Todo` queue. This path creates a new `Todo` issue with the `Claimed` label for issues that are already beyond triage quality and ready to enter planning immediately from chat context. It never makes code changes. Then read [references/direct-planning-issue-path.md](references/direct-planning-issue-path.md).
- `Triage Path`: Use when the task is a backlog-refinement session for fuzzy accepted ideas. This path fetches the first 7 `Draft` issues, shapes them into clearer problem/value/scope/solution writeups, gets user approval, and moves accepted issues to `Triaged`. It never makes code changes. Then read [references/triage-path.md](references/triage-path.md).
- `Planning Path`: Use when the task is feature planning for the top unclaimed issue in `Todo`. This path claims that issue by adding the `Claimed` label, sizes the issue into `Fast`, `Standard`, or `Deep` planning depth, studies the codebase and repository conventions, works with the user only on the decisions that materially matter, rewrites the Linear description into a full implementation-ready spec, and may replace `Claimed` with `Ready` when planning is approved. It never makes code changes. Then read [references/planning-path.md](references/planning-path.md).
- `Implementation Path`: Use when the task is to start building a `Ready`-labeled issue from `Todo`. If this chat just finished planning an issue and the user asks to implement, continue with that same issue instead of selecting the top `Ready` issue from the queue. Otherwise, select the next `Ready` issue by manual queue order. This path gets user approval, moves the issue to `In Progress`, removes the `Ready` label, branches into code-execution or admin-execution behavior based on the issue kind and the real work required, follows repository planning and documentation rules where applicable, supports user testing, and completes delivery through commit-flow when code work exists. Then read [references/implementation-path.md](references/implementation-path.md).

## Default Path

If the chat is fresh and the user asks to use this skill without further routing instructions, default to the `Triage Path`.

## Shortcut Routing Examples

Treat short user requests like these as explicit path selection, not as ambiguous wording:

- `use linear flow to triage` -> `Triage Path`
- `triage using linear flow` -> `Triage Path`
- `use linear flow for assertions` -> `Assertion Path`
- `validate assertions using linear flow` -> `Assertion Path`
- `use linear flow to validate assertions` -> `Assertion Path`
- `create a triaged issue using linear flow` -> `Direct Triaged Issue Path`
- `create a triaged issue directly using linear flow` -> `Direct Triaged Issue Path`
- `add this as a triaged issue using linear flow` -> `Direct Triaged Issue Path`
- `create a planning issue using linear flow` -> `Direct Planning Issue Path`
- `add this directly to planning using linear flow` -> `Direct Planning Issue Path`
- `turn this discussion into a planning issue using linear flow` -> `Direct Planning Issue Path`
- `create a planned issue from this discussion using linear flow` -> `Direct Planning Issue Path`
- `use linear flow to plan` -> `Planning Path`
- `plan using linear flow` -> `Planning Path`
- `use linear flow to implement` -> `Implementation Path`
- `implement using linear flow` -> `Implementation Path`
- `use linear flow for discovery` -> `Discovery Path`
- `discover using linear flow` -> `Discovery Path`

If the user uses one of these verbs clearly, do not ask which path they mean.

## Guardrails

- Do not make code changes in Discovery, Triage, or Planning.
- Do not use Linear comments for pre-implementation shaping. Keep that work in the issue description.
- Use comments only during implementation and after, or when canceling an issue and recording the reason.
- Use real newline characters in Linear content. Do not write literal `\n` or `\\n`.
- Use valid Markdown in every Linear description update.
- Do not place template-based user-facing deliverables inside fenced code blocks unless the user explicitly asks to see the raw Markdown.

# Implementation Path

Use this path to start and complete a `Ready`-labeled implementation issue from Linear.

Use `planning/linear.md` as the default source for team and project identifiers when that file exists. Do not spend Linear calls rediscovering the authenticated user, team list, project list, or status list if `planning/linear.md` already provides the needed identifiers for this implementation session.

## Workflow

1. Select the implementation issue using this precedence order:
   - If the user explicitly names an issue, use that specific issue after verifying it is in `Todo` and has the `Ready` label.
   - If the current chat has just planned or finalized a single issue and the user asks to implement, continue with that same issue after verifying it is in `Todo` and has the `Ready` label.
   - Treat the issue from the most recent Planning Path or Direct Planning Issue Path in this chat as the selected issue, even if other `Ready` issues exist above it in the manual queue.
   - In those specific-issue cases, do not run the ordered GraphQL queue-selection query and do not ask which `Ready` issue is top of the queue. The user's same-chat implementation request is scoped to the issue already being discussed.
   - Only when there is no explicit issue and no clear current-chat planned issue, take the first `Ready`-labeled issue from `Todo` using the canonical queue-selection rule.
   - For queue-selection only, read `planning/linear.md` first and use its cached project id and `Ready` label id when available.
   - Use actual Linear MCP/app tools such as `get_issue`, `update_issue`, and `list_issue_labels` for normal issue reads and updates; do not invent tool names.
   - For queue-selection only, if the `Ready` label id is missing or stale, fetch the team labels once, confirm the `Planning` group plus `Claimed` and `Ready` child labels, and update `planning/linear.md`.
   - For queue-selection only, use the direct Linear GraphQL API to query `Todo` issues with the `Ready` label, `first: 1`, `identifier`, `title`, and `sortOrder`.
   - For queue-selection only, filter the queue query with `labels: { id: { eq: $readyLabelId } }`.
   - For queue-selection only, include `sort: [{ manual: { order: Ascending } }]` in the GraphQL query.
   - For queue-selection only, treat the returned order as the real manual queue order among implementation-ready issues.
   - If a required queue-selection query cannot be completed authoritatively, stop immediately and tell the user in `final`; do not continue implementation selection with any fallback ordering.
2. Do not move the issue to `In Progress` yet. First read it with the Linear MCP/app and prepare the approval gate.
3. If the issue has gone stale, refresh it before implementation.
   - Treat an issue as stale when it has not been updated for 14 or more days, or when the relevant codebase area has materially changed since the `Ready` label was applied.
   - Revalidate the plan before coding. If the issue is no longer implementation-ready, stop and ask the user whether it should go back for further planning.
4. Read the repository rules, planning docs, and `AGENTS.md` if they have not already been read in the current session.
5. Determine the execution mode before doing any implementation work:
   - `Code Execution Mode`: use when the issue work is primarily repository changes
   - `Admin Execution Mode`: use when the issue is labeled `Admin` and the work is primarily external configuration, dashboard, provider, policy, process, or other operational work
   - if the issue is labeled `Admin` but still clearly requires repository changes, use a mixed approach: start in `Admin Execution Mode` to frame the real work, then enter `Code Execution Mode` only for the code portion
6. Send one consolidated approval-gate message in `final` so it remains the visible user-facing deliverable.
7. That message must include the issue summary, execution mode, what will change, the implementation approach, the expected documentation updates, and an explicit approval request.
8. After sending that message, stop. Do not send a follow-up message such as `waiting for approval`, and do not make further tool calls until the user responds.

## Approval Gate Message

Use this exact shape for the approval-gate message in `final`:

```markdown
# {Issue Title}

## 📋 Issue

{Issue summary}

## 🛠️ Execution Mode

{Code Execution Mode | Admin Execution Mode | Mixed}

## 🔧 What Will Change

- Change 1
- Change 2

## 🛠️ Implementation Approach

{How the implementation will be done}

## 📚 Documentation Impact

- Doc update 1
- Doc update 2
- Or `None.`

## ✅ Approval Request

Approve this implementation plan and documentation scope before any code changes begin.
```

This must be the last message before stopping for approval.

For `Admin` issues, `## What Will Change` and `## Implementation Approach` must describe the real operational work. Do not imply repository code changes unless they are actually part of the approved scope.

## Execution Mode Rule

- `Code Execution Mode` means repository implementation is the main work. Follow the normal worktree, branch, testing, and commit flow.
- `Admin Execution Mode` means the main work is external to the repository. The agent's role is to guide, verify, document, and coordinate that implementation with the user instead of assuming direct code changes.
- `Mixed` means both are required. Start by making the non-code and code boundaries explicit to the user, then apply the relevant parts of each mode deliberately.

## Branch And Worktree Flow

After the user approves implementation:

1. Move the issue to `In Progress` and remove the `Ready` label before implementation work begins.
2. Do not remove normal kind labels such as `Feature`, `Improvement`, `Bug`, or `Admin`.
3. If the approved work includes repository code changes, record the current branch as the parent branch.
4. If the approved work includes repository code changes, check whether the current session is already inside a dedicated worktree.
5. If the session is already inside a worktree:
   - create the dedicated issue branch in that worktree immediately, before documentation edits, code edits, testing setup, or any other implementation work begins
   - do this as soon as approval is granted so the user can immediately see the issue branch in the IDE for that worktree
   - continue the code portion there only after the branch exists
6. If the session is not inside a worktree and code changes are required:
   - stop before making code changes
   - ask the user to hand off the chat into a new worktree using the Codex UI
   - resume only after the chat is running inside that worktree
   - create the dedicated issue branch there immediately before any implementation work begins
7. If the approved work does not include repository code changes, do not require a worktree just to perform admin-only execution.

## Worktree Execution Note In AGENTS.md

Once implementation is confirmed to be running inside a dedicated worktree:

- Use the repository `AGENTS.md` file as a transient, worktree-local execution notebook for this implementation only.
- Add a clearly marked temporary note block near the end of `AGENTS.md` with unique start and end markers so it can be updated and removed safely.
- That temporary note should include at least:
  - current issue id and title
  - current path: `Implementation Path`
  - current worktree path
  - current issue branch
  - current implementation checkpoint
  - brief progress summary
  - immediate next step
  - a reminder to refresh the `linear-flow` skill entry and `implementation-path.md` after compaction before continuing
  - a reminder that before closeout and `commit-flow`, this temporary note must be removed from `AGENTS.md`
- Update that temporary note whenever the implementation checkpoint changes materially so future resumptions have an accurate state snapshot.
- Treat the note as an instruction to future runs in the same worktree after compaction: resume the active `Implementation Path`, refresh the relevant `linear-flow` instructions, reconcile the current Linear issue state, and then continue from the recorded checkpoint instead of drifting into ad hoc implementation.
- Do not use this temporary note outside an active implementation running in a worktree.
- Do not create this temporary note for admin-only execution that does not use a worktree.

## Worktree Rule

- Worktree creation is user-controlled and must be done in the Codex UI, not by the agent.
- The agent may detect the current workspace shape and may create branches inside an existing worktree.
- The agent must never create a new worktree on its own for this workflow.
- The agent must never create a temporary clone, copied repository, copied worktree, ad hoc backup workspace, or any other second working copy outside the active tracked workspace for this workflow.
- The agent must never move files out of the active worktree into another temporary directory in order to continue implementation, testing, closeout, commit, push, or merge work.
- All implementation, verification, closeout, commit-flow, and merge preparation work must stay inside the active tracked workspace or the parent workspace already associated with the workflow.
- If the active worktree or workspace has a problem that prevents safe continuation, stop immediately and ask the user what to do. Do not improvise by cloning, copying, or exporting the repository elsewhere.

## Environment Files In Worktrees

- Git worktrees often do not have the parent repository's `.env` files present locally.
- Before running any env-dependent command in a worktree, first determine the parent repository root outside the worktree.
- Treat commands such as `npm run dev`, `npm run build`, and other build, test, or runtime commands that rely on validated environment variables as env-dependent.
- Do not stop just because the worktree does not contain `.env` files. Source the parent repository env files into the shell for that command instead.
- Prefer this exact shell pattern:

```bash
bash -lc 'set -a
[ -f "/absolute/path/to/parent-repo/.env" ] && source "/absolute/path/to/parent-repo/.env"
[ -f "/absolute/path/to/parent-repo/.env.local" ] && source "/absolute/path/to/parent-repo/.env.local"
[ -f "/absolute/path/to/parent-repo/.env.development" ] && source "/absolute/path/to/parent-repo/.env.development"
[ -f "/absolute/path/to/parent-repo/.env.development.local" ] && source "/absolute/path/to/parent-repo/.env.development.local"
set +a
npm run dev'
```

- Replace `npm run dev` with the actual env-dependent command when needed, for example `npm run build`.
- Use the same sourcing pattern for any other env-dependent operation that needs the parent repository configuration.
- Do not copy the env files into the worktree.
- If the parent repository root is unclear, determine it explicitly before proceeding instead of guessing.

## Documentation Gate

If repository documentation updates are required before implementation starts, especially planning or architecture docs:

1. Make those documentation updates first in the issue worktree.
2. Show the user the documentation changes.
3. Wait for explicit approval of the documentation changes before continuing into code changes.

For admin-only execution, apply the same principle to any pre-execution documentation or runbook updates that the repository or the user expects before the operational steps begin.

## Implementation Standard

- Follow the approved plan fully.
- Run your own tests and verifications along the way.
- Keep going until the issue is actually resolved, not just partially addressed.
- Use repository-specific rules and required skills as appropriate for the stack in that repository.
- If a true blocker appears after implementation has started, move the issue to `Blocked` and explain the blocker clearly to the user.
- If the issue is `Admin`, default to guidance, verification, and coordination work first rather than assuming code edits are the main implementation mechanism.
- For `Admin` issues, clearly distinguish:
  - steps the agent can perform directly
  - steps the user must perform in external dashboards, providers, or accounts
  - any optional repository follow-up work
- Do not invent alternative repository flows during implementation or closeout. Stay inside the tracked workflow defined here.
- If a merge, commit-flow step, worktree state, or filesystem situation becomes unclear, stop and ask the user instead of creating any temporary clone, copied workspace, or side directory.

## User Testing Handoff

After implementation is complete:

1. Make the work easy for the user to test manually.
2. Move the issue to `In Review` when the implementation is complete and waiting for the user's testing and approval.
3. If the implementation includes a web application change that needs local validation, start the dev server on one of the user-approved ports only: `3000`, `3001`, `3002`, `3003`, `3004`, or `3005`.
4. Prefer the first available port in that approved range and pass it explicitly to the dev server command.
5. If all six approved ports are already in use, stop and ask the user in `final` which port should be used instead. Do not pick another port on your own.
6. Track whether this session actually started the dev server process. Only the agent that started that process may later terminate it.
7. Never terminate a dev server that this session did not start itself, even if it is using one of the approved ports. Another agent may be using that server from another worktree.
8. Provide the testing entry URL using the approved port that was actually used when local app testing is relevant.
9. If this is an admin-only issue, provide the easiest realistic validation path for the user instead of forcing a dev server step.
10. If it is another kind of project, provide the easiest realistic test path.
11. Wait for the user's testing and review.

## Completion Flow

Once the user accepts the implementation:

1. If there is a known non-blocking annoyance, small bug, integration limitation, or residual issue that should be tracked later, ask the user whether they want a follow-up issue created for it.
2. If the user wants that follow-up tracked, create it through the `Discovery Path` as a new `Suggested` issue before closing the current implementation issue.
3. Treat that follow-up as a separate issue for later review. Do not expand the current implementation scope just because the follow-up issue was created.
4. If repository changes were made, before starting the `Pre-Commit Mergeability Probe` or using the `commit-flow` skill, remove the temporary implementation note block from `AGENTS.md` so it is not included in the probe or committed with the feature work.
5. If repository changes were made, run the `Pre-Commit Mergeability Probe` below before using the `commit-flow` skill.
6. If repository changes were made and the probe passes, use the `commit-flow` skill to commit the full worktree branch, push it, and create a GitHub PR targeting the recorded parent branch.
7. Before considering code delivery complete, make sure the PR description uses the native Linear linking rule below so the PR is linked to the issue through the GitHub/Linear integration.
8. Add the closeout comment using the template below.
9. Move the issue to `Done` manually after the closeout comment is added. Do not wait for PR merge automation to move the issue.
10. If repository changes were made, leave the branch and PR as the delivery artifact. Do not merge the issue branch back into the parent branch locally during this flow.
11. If the issue was admin-only and no repository changes were required, skip the worktree, mergeability probe, PR, and commit-flow steps entirely.
12. Do not perform any closeout step from a temporary clone, copied worktree, or other untracked directory.
13. If the active worktree or parent workspace cannot safely complete closeout as written here, stop immediately and ask the user for instructions.

## Pre-Commit Mergeability Probe

Run this check after the user accepts the implementation and before creating the first real commit for repository changes.

Purpose:

- Check whether the current dirty worktree, if committed as-is, would merge cleanly into the latest remote parent branch.
- Catch conflicts caused by parallel agents before `commit-flow` creates a real commit.
- Keep the probe non-invasive: no real commit, no branch movement, no real index changes, no temporary clone, and no copied worktree.

Required setup before running the probe:

- Confirm the parent branch name recorded at implementation start.
- Remove the temporary implementation note block from `AGENTS.md`.
- Run `git status --short` and verify the dirty worktree contains only intended issue changes.
- If unintended files are present, stop and ask the user before probing or committing.

Use this exact command pattern from the active issue worktree:

```bash
parent_branch="<parent-branch>"

git fetch origin "$parent_branch"

tmp_index="$(mktemp)"
trap 'rm -f "$tmp_index"' EXIT

GIT_AUTHOR_NAME="Codex Mergeability Probe" \
GIT_AUTHOR_EMAIL="codex-mergeability-probe@example.invalid" \
GIT_COMMITTER_NAME="Codex Mergeability Probe" \
GIT_COMMITTER_EMAIL="codex-mergeability-probe@example.invalid" \
bash -lc '
  set -e
  GIT_INDEX_FILE="$0" git read-tree HEAD
  GIT_INDEX_FILE="$0" git add -A
  tree="$(GIT_INDEX_FILE="$0" git write-tree)"
  probe_commit="$(printf "pre-commit mergeability probe\n" | git commit-tree "$tree" -p HEAD)"
  git merge-tree --write-tree --quiet "$probe_commit" "origin/$1"
' "$tmp_index" "$parent_branch"
```

Interpretation:

- Exit code `0`: the dirty worktree snapshot should merge cleanly into `origin/<parent-branch>`. Continue with the `commit-flow` skill.
- Nonzero exit code: do not create a real commit yet. Gather conflicting paths and stop for user guidance.

To gather conflict paths after a nonzero result, rerun the same synthetic commit setup and replace the final command with:

```bash
git merge-tree --write-tree --name-only "$probe_commit" "origin/$1"
```

On conflict:

- Stop before committing.
- Report the conflicting paths.
- Recommend rebasing the issue branch onto `origin/<parent-branch>` in the active worktree, resolving conflicts, rerunning validation, and rerunning this pre-commit probe.
- Do not rebase, merge, or resolve conflicts automatically unless the user explicitly approves that recovery step.

## Native Linear PR Linking Rule

- When implementation delivery includes creating or updating a GitHub PR and native Linear/GitHub integration is available, prefer native PR linking through the PR description instead of manually attaching the PR URL to the Linear issue.
- Use only these two Linear magic-word forms in this workflow:
  - `Part of WAL-84` for non-closing support links
  - `Resolves WAL-84` for closing links
- If the PR is intended to close one or more implementation issues, add one `Resolves` line per issue in the PR description, for example:
  - `Resolves WAL-83`
  - `Resolves WAL-84`
  - `Resolves WAL-85`
- Use the same one-line-per-issue pattern for multi-issue PRs. Do not collapse multiple issue identifiers onto a single ambiguous line when clarity would suffer.
- If the PR is only linking supporting work and is not intended to close an issue, use one `Part of` line per issue instead.
- Non-closing magic words create the native PR link but do not make merge move the issue to `Done`. Do not assume merge automation will close that issue.
- Closing magic words both create the native PR link and allow merge-time Linear automation to move those issues to `Done` when that integration is configured.
- If the branch name, PR title, commit message, and PR description disagree, treat the PR description as the authoritative place to make the intended close-or-link behavior explicit.
- Do not rely on manual PR URL attachments as the primary linkage mechanism when native Linear/GitHub linking is available.
- Even when `Resolves` is used, still move the Linear issue to `Done` manually in the Completion Flow after the user accepts the implementation and the closeout comment is added.

## Linear Writing Rule During Implementation

- Keep pre-implementation planning in the issue description.
- Use comments for implementation closeout, test notes, cancellation reasons, and other execution-stage updates.

## Ready Label Description Rule

A `Ready`-labeled `Todo` issue reuses the finalized planning description unchanged. Do not rewrite the description when implementation starts unless the user explicitly wants content changes.

## Closeout Comment Template

Use this closing comment before moving an issue from `In Review` to `Done`:

```markdown
# {Issue Title}

## 🚀 What Shipped

- Shipped change 1
- Shipped change 2

## 🧪 Validation

- Validation step 1
- Validation step 2

## 📚 Docs Updated

- Updated doc 1
- Updated doc 2

## 🔁 Notable Follow-up

- Follow-up item 1
- Follow-up item 2

## 🧷 Commit

- `commit-sha` Commit subject
```

Rules:

- Use real Markdown line breaks.
- If there were no docs updates, write `- None.`
- If there is no notable follow-up, write `- None.`
- If a `Suggested` follow-up issue was created for a residual problem, reference it in `## Notable Follow-up`.
- Include the final commit SHA and a concise commit subject when a commit exists.
- If there was no repository commit because the issue was admin-only, write `- None.` under `## Commit`.
- Standardize around the closeout style already used in TweetWizard `Done` issues such as `WAL-15`, `WAL-16`, and `WAL-17`.

## Promotion Gates

Only move a `Ready`-labeled `Todo` issue to `In Progress` when:

- the issue is the selected implementation target as determined by the rules above
- the user has approved the implementation plan and documentation scope
- the `Ready` label is removed at the same time the issue is moved to `In Progress`

Only move `In Progress -> In Review` when:

- the implementation work is complete enough for user testing
- the intended docs and code changes for the approved scope are in place when code changes were part of the plan
- the intended operational or administrative work is complete enough for user review when the issue is admin-led
- the user can exercise the feature through a clear testing path

Only move `In Review -> Done` when:

- the user has accepted the work
- the closeout comment has been added in the required format
- any notable accepted gaps or follow-ups are recorded

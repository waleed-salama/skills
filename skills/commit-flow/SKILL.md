---
name: commit-flow
description: "Analyze a repository diff, audit the documentation affected by that diff, update stale documentation before staging, and then deliver the changes in one of three modes: only commit, commit and push, or commit, push, and open a ready-for-review GitHub pull request. Use when Codex is asked to prepare a commit or complete a Git delivery flow and the commit must reflect the full change set with current documentation."
---

# Commit Flow

## Overview

Use this skill when the user wants Codex to perform a Git delivery step and expects commit preparation to include documentation hygiene. Always review the full diff first, audit the documentation that should describe the changed behavior or structure, update stale documentation before committing, and make the commit message describe the entire staged change set.

When the requested delivery is a production release from `dev` to `main` or `master`, use the Production Release Path instead of the normal feature-branch PR path. In that path, update `changelog.md` on `dev`, commit that changelog update directly to `dev`, push `dev`, and open the production PR from `dev` to `main` or `master`.

## Delivery Modes

Choose exactly one mode before executing Git commands:

- `only-commit`: stage and create a commit, then stop.
- `commit-and-push`: stage, commit, and push the current branch.
- `commit-push-pr`: stage, commit, push, and open a ready-for-review PR with `gh`.

If the user does not specify a mode, ask for one short clarification or make the smallest safe assumption from the request wording.

## Documentation Audit Rule

Treat documentation review as a required diff-to-doc audit, not as a file-existence check.

For every commit flow:

1. Identify the durable changes in the diff.
   - Focus on user-visible behavior, architecture, workflows, setup, commands, structure, design-system policy, APIs, schemas, and operating conventions.

2. Identify which documentation should describe those changes.
   - Common examples include `README`, setup docs, architecture docs, planning docs, project-structure docs, rules/conventions, API/schema docs, migration notes, or agent/session guidance.
   - Use the repo's own documentation layout rather than assuming fixed filenames.

3. Open the relevant docs and compare their actual statements with the code and structure in the diff.
   - Do not stop after noticing that a doc exists.
   - Verify that the doc is still true after the change.

4. Update stale docs before staging the final commit.
   - If the change introduced a durable new concept and no existing doc covers it, add or extend the most appropriate existing doc.
   - Keep the documentation update in the same change set as the code change.

5. If no documentation update is needed, be able to state a concrete reason.
   - The reason must reference the kind of change made and why it does not affect any durable documented behavior, structure, setup, workflow, or convention.

## Workflow

1. Inspect repository state.
   - Run `git status -sb`.
   - Review the full change set, not just the latest chat step.
   - If needed, inspect staged and unstaged diffs separately.
   - Determine whether this is a normal delivery or a production release from `dev` to `main` or `master`.

2. Audit documentation.
   - Enumerate the changed areas from the diff.
   - Map each changed area to the documentation that should describe it.
   - Open and verify the relevant docs.
   - Update any stale documentation before continuing.
   - Do not proceed to commit generation until this audit is complete.

3. Validate the final scope.
   - Confirm the commit will reflect the intended complete change set.
   - Avoid sweeping unrelated work into the commit unless the user asked for that.
   - If the worktree contains unrelated user changes, do not revert them.

4. Generate the commit description.
   - Analyze the actual staged diff.
   - Write the commit message in two parts:
     - First line: a concise, fully descriptive subject that reflects the entire commit.
     - Blank line.
     - Body: a concise flat bullet list that summarizes the important changes, features, fixes, migrations, schema updates, dependencies, or files/modules touched.
   - Construct the final commit message with real newline characters. Do not pass a shell string that contains literal `\n` sequences and assume Git will render them as line breaks.
   - Prefer one of these safe patterns when executing the commit:
     - multiple `-m` flags for subject and body paragraphs when each `-m` value is already real multiline text
     - `git commit -F <file>` with a temp file
     - `git commit -F -` with a heredoc or stdin pipe
   - The subject and bullets must reflect the entire commit, not just the last action in the conversation.
   - If the diff spans multiple concerns, summarize the real combined outcome rather than naming one subtask.
   - Keep the body high signal. Prefer bullets that capture durable changes such as user-visible behavior, major implementation pieces, data model changes, dependency upgrades, and important file or route additions.
   - Do not write a one-line-only commit unless the user explicitly asks for that format.

5. Run checks.
   - Run relevant validation if it has not already been run after the final edits.
   - If checks fail due to missing dependencies or tools, install only if appropriate and permitted, then rerun once.

6. Execute the selected mode.

## Production Release Path

Use this path when the user asks to release, promote, or create a production PR from `dev` to `main` or `master`, or when the intended PR head is `dev` and the base is `main` or `master`.

This path is an explicit exception to the normal branch rule:

- Do not create a new `codex/*` branch.
- Do not move the changelog update onto a feature branch.
- Work on `dev`, commit the changelog update to `dev`, push `dev`, then open the PR from `dev` to the production branch.

Before editing:

1. Confirm the current branch is `dev`, or switch to `dev` only if the user clearly requested the production release and the worktree state allows it without disturbing unrelated changes.
2. Identify the production base branch, preferring the user's requested base, then `main`, then `master`.
3. Fetch the relevant remote refs.
4. Review the aggregate release scope from production to `dev`:
   - inspect commit subjects and bodies in `origin/<base>..HEAD`;
   - inspect the combined diff with `origin/<base>...HEAD`;
   - include already-merged feature and fix commits, not just the latest local change.

Update `changelog.md` before committing:

- Prefer an existing root-level `changelog.md`; if the repo uses `CHANGELOG.md`, update that file instead. If no changelog exists, create a root-level `changelog.md`.
- Add a dated release entry for the production PR.
- Write for users, not developers.
- Summarize the visible value delivered since the current production branch: new capabilities, workflow improvements, bug fixes, reliability improvements, and UX changes.
- Avoid implementation details, internal file names, branch names, dependency names, schema details, and low-level optimization notes unless users directly experience them.
- Group changes under plain-language headings such as `Added`, `Improved`, and `Fixed` when that makes the release easier to scan.
- If a commit is purely internal and has no user-visible impact, omit it or fold it into a user-facing reliability or maintenance note only when appropriate.

Then execute the selected production release mode:

- Stage the changelog update and any explicitly intended release-prep files.
- Commit directly on `dev` with a message that clearly describes the changelog update for the production release.
- Push `dev` with tracking: `git push -u origin dev`.
- For `commit-push-pr`, open a ready-for-review PR from `dev` to the production base branch.
- Generate the PR title and body from the full `dev` to production delta.

For a production release PR, do not use the normal single-issue PR body template. A release PR is an aggregate promotion, so it should not force `User-visible issue` or `Root cause` sections unless the release genuinely centers on one incident.

Use this release PR title shape:

```text
Release dev to <base>
```

Use this release PR body shape:

```text
## Summary
- Promote the accumulated changes from `dev` to `<base>`.
- Include the dated changelog update for this release.

## Release highlights
- Summarize the major features, improvements, and fixes included in the full `dev` to `<base>` delta.
- Group related changes when helpful.

## Operational notes
- Note migrations, environment changes, rollout considerations, known risks, or follow-up work.
- Write `None` if there are no operational notes.

## Documentation
- Mention the changelog update and any other docs changed for the release.

## Validation
- List the checks run for the release.
```

The release PR body is for reviewers and maintainers, not end users. It can mention branches, validation, docs, operational risks, and technical coordination details where useful. Keep the changelog entry as the user-facing release summary.

### `only-commit`

- Stay on the current branch.
- Stage the intended files with `git add -A` unless the user requested narrower staging.
- Commit with the generated message using real newlines, not literal `\n` escape sequences.
- Stop after confirming the commit.

### `commit-and-push`

- For a production release from `dev` to `main` or `master`, use the Production Release Path instead.
- If currently on `main`, `master`, or the repo default branch, create `codex/{description}` first.
- Otherwise stay on the current branch.
- Stage the intended files with `git add -A` unless the user requested narrower staging.
- Commit with the generated message using real newlines, not literal `\n` escape sequences.
- Push with tracking: `git push -u origin $(git branch --show-current)`.

### `commit-push-pr`

- Require GitHub CLI `gh`. Check `gh --version`.
- Require authenticated `gh`. Check `gh auth status`.
- For a production release from `dev` to `main` or `master`, use the Production Release Path instead.
- If currently on `main`, `master`, or the repo default branch, create `codex/{description}` first.
- Otherwise stay on the current branch.
- Stage the intended files with `git add -A` unless the user requested narrower staging.
- Commit with the generated message using real newlines, not literal `\n` escape sequences.
- Push with tracking: `git push -u origin $(git branch --show-current)`.
- Determine the PR base branch.
- Review the full PR scope using both commit history and combined diff, not just `HEAD`:
  - inspect the commits that are on the branch and not on the base branch,
  - inspect the combined file diff from `base...HEAD`.
- Generate the PR title and body from that full branch-to-base delta.
- The latest commit message may inform the PR title, but it must not be the only source.
- Open a ready-for-review PR with a title that reflects the full PR diff, usually `[codex] {description}`.
- Write the PR body to a temp file with real newlines before calling `gh pr create`.
- In the PR body, cover the user-visible issue, root cause, what changed across the full PR, the documentation updates, and the checks used to validate the work.

## Guardrails

- Do not commit before the documentation audit is complete.
- Do not treat documentation review as satisfied just because the repo has a `README` or a planning file.
- Do not generate a commit message from memory or from the last chat step alone.
- Do not omit the commit body when using the default `commit-flow` format.
- Do not build the commit body by embedding literal `\n` escape sequences in a shell string; use real newlines via stdin, a heredoc, a temp file, or a correctly formed multiline `-m` value.
- Do not amend or rewrite history unless the user explicitly asks.
- Do not revert unrelated user changes.
- Do not create a feature branch for a production release whose PR should run from `dev` to `main` or `master`.
- Do not write production changelog entries as technical commit logs; write them as user-facing release notes.
- Do not open a PR in `only-commit` or `commit-and-push` mode.
- Do not require a PR when the user only asked for a commit or a push.

## Example Requests

- `Use $commit-flow to only commit these changes.`
- `Use $commit-flow to commit and push the current work.`
- `Use $commit-flow to commit, push, and open a PR.`
- `Use $commit-flow to prepare the production PR from dev to main.`
- `Use $commit-flow before committing and make sure the docs are actually up to date.`

## Commit Message Shape

Default to this format unless the user explicitly requests another style:

```text
Concise subject describing the full change

- Bullet summarizing a major change
- Bullet summarizing another major change
- Bullet noting important migrations, schema updates, or dependency changes
```

Examples:

```text
Upgrade the repo to Next.js 16.2.6 with Tailwind v4

- Upgrade Next.js to v16.2.6 and rename `middleware.ts` to `proxy.ts`
- Upgrade Tailwind to v4 and remove obsolete Tailwind dependencies
```

```text
Add feature X

- Add the new `/app/feature-page` route for feature X
- Add `serverActionName` to support the new workflow
- Add database table `table-name` and field `fieldname`
- Add dependency `dependency-name`
```

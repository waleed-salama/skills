# Assertion Path

Use this path to validate the top `Suggested` issue labeled `Assertion` without routing it through the normal backlog flow of `Draft -> Triaged -> Todo`, followed by `Claimed` and `Ready` planning labels.

## Scope

- Use this path only for issues labeled `Assertion`.
- `Assertion` issues belong in `Suggested`, not `Draft`.
- By default, assertion issues stay deferred in `Suggested` until the user intentionally starts this path.
- This path may require repository work if validation should be expressed as automated coverage or if the assertion must be actively exercised in code.
- If `planning/linear.md` exists, use it first for team and project identifiers instead of rediscovering them with extra Linear calls.

## Assertion Semantics

- An assertion is a fear, edge case, pitfall, or expected-behavior claim that should be validated.
- It is not normal product triage work.
- It is not required to have one of the normal kind labels while it remains an assertion-only issue.
- If validation reveals real product work that is missing or broken, create a separate normal issue for that finding instead of pretending the assertion itself was a feature request.

## Workflow

1. Take the first issue from the `Suggested` queue that is labeled `Assertion`.
   - read `planning/linear.md` first and use its cached project id when available
   - use actual Linear MCP/app tools such as `get_issue` and `update_issue` for normal issue reads and updates; do not invent tool names
   - use the direct Linear GraphQL API to query `Suggested` issues with `first: 1`, `identifier`, `title`, and `sortOrder`
   - filter that query to only issues carrying the `Assertion` label
   - include `sort: [{ manual: { order: Ascending } }]` in the GraphQL query
   - treat the returned order as the real manual queue order
   - if this ordered query cannot be completed authoritatively, stop immediately and tell the user in `final`; do not continue with any fallback ordering
2. Read the issue description in full with the Linear MCP/app and restate the assertion clearly.
3. Determine the validation mode:
   - `Automated Coverage Mode`: use when the relevant part of the project already has meaningful automated tests or clear testing conventions that can absorb this assertion as a test scenario
   - `Manual Validation Mode`: use when meaningful automated testing does not exist yet for that area
4. Send one consolidated approval-gate message in `final` before doing validation work. That message must include:
   - the assertion refresher
   - the chosen validation mode
   - whether repository changes are expected
   - the validation approach
   - the explicit approval request
5. After sending that message, stop. Do not send a follow-up `waiting` message, and do not make further tool calls until the user responds.
6. After approval:
   - in `Automated Coverage Mode`, validate the assertion by adding or updating automated coverage first
   - in `Manual Validation Mode`, validate the assertion through code review, direct functional checks, or other realistic validation that fits the project
7. If repository changes are required:
   - follow the same worktree, branch, documentation, and testing discipline used by the `Implementation Path`
   - if the assertion work becomes meaningful repository implementation, reuse the implementation conventions rather than inventing a looser standard
8. Once validation is complete, branch the outcome:
   - `Pass`: the assertion is now sufficiently validated
   - `Fail`: the assertion exposed a real gap, bug, regression, missing coverage need, or product-work item

## Approval Gate Message

Use this exact shape for the approval-gate message in `final`:

```markdown
# {Issue Title}

## 📋 Issue

{Assertion summary}

## 🛠️ Validation Mode

{Automated Coverage Mode | Manual Validation Mode}

## 🔧 What Will Change

- Change 1
- Change 2
- Or `None.` if validation is expected to be read-only

## 🛠️ Implementation Approach

{How the assertion will be validated}

## 📚 Documentation Impact

- Doc update 1
- Doc update 2
- Or `None.`

## ✅ Approval Request

Approve this assertion-validation plan before validation work begins.
```

This must be the last message before stopping for approval.

## Validation Standard

When validating an assertion:

- start from the exact fear or behavior claim recorded in Linear
- define what counts as a pass
- prefer reproducible validation over vague confidence
- if automated testing is already part of the relevant project area, prefer encoding the assertion as coverage rather than leaving it as a manual check
- if automated testing is not meaningfully present in that area, do not force the repository into premature test architecture just to satisfy the assertion

## Assertion Outcome Rules

If the assertion passes:

- add a closeout comment summarizing how it was validated
- move the issue to `Done`

If the assertion fails:

- do not move the assertion issue to `Done` as if it passed
- create a separate normal issue for the discovered gap
- classify that new issue with the appropriate normal kind label: `Bug`, `Improvement`, `Feature`, or `Admin`
- put that new issue into the appropriate intake stage based on maturity:
  - `Draft` if it is still fuzzy backlog work
  - `Triaged` only if the issue is already clear enough to meet normal triage standards
- update the assertion issue to reference the follow-up issue
- then move the assertion issue to `Done` only as a completed validation record, not as proof that the product behavior was already correct

## Closeout Comment Template

Use this closeout comment before moving a validated assertion to `Done`:

```markdown
# {Issue Title}

## 🚀 What Shipped

- Added assertion coverage, or `None.`
- Functional changes made during validation, or `None.`

## 🧪 Validation

- Validation method 1
- Validation method 2

## 📚 Docs Updated

- Updated doc 1
- Or `None.`

## 🔁 Notable Follow-up

- Follow-up issue reference if the assertion exposed a gap
- Or `None.`

## 🧷 Commit

- `commit-sha` Commit subject
- Or `None.` if validation was read-only
```

## Guardrails

- Do not send assertion issues into the normal `Draft` triage flow.
- Do not auto-convert every assertion into a bug.
- Do not mark a failed assertion as a successful pass.
- If validation becomes substantial repository work, use the same quality bar as the normal `Implementation Path`.

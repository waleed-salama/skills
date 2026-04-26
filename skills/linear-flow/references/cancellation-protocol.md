# Cancellation Protocol

Read this file only when an issue is about to be moved to `Canceled` or `Duplicate`.

## Rules

- Do not cancel or mark as duplicate silently.
- Prefer explicit user approval before moving an issue into a terminal non-completed state.
- Keep the pre-implementation description intact unless the user wants it rewritten.
- Add the cancellation or duplicate reason as a comment, not in the description.
- Ask for that approval in one visible `final` message, then stop. Do not send a trailing message such as `waiting for cancellation approval`, and do not make further tool calls until the user responds.

## Canceled Comment Template

```markdown
# {Issue Title}

## Reason for Cancellation

Why this issue is being canceled.

## Context

Relevant context for why the work is no longer needed, no longer valid, or no longer worth pursuing.

## Follow-up

- Replacement issue, if any
- Deferred future revisit, if any
```

## Duplicate Comment Template

```markdown
# {Issue Title}

## Reason for Duplicate

Why this issue is considered a duplicate.

## Canonical Issue

Reference the issue that should be used instead.

## Notes

Any context that helps future readers understand the merge of intent.
```

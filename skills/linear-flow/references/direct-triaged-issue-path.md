# Direct Triaged Issue Path

Use this path when the user already discussed an issue in chat and wants a new Linear issue created directly in `Triaged`.

## Scope

- Do not make code changes.
- Keep all work in chat and the Linear issue description.
- Do not route this intake through `Suggested` or `Draft`.
- Use this path only when the user wants to create a brand-new issue directly in `Triaged`.
- Use `planning/linear.md` as the default source for team and project identifiers when that file exists.
- Do not spend Linear calls rediscovering the authenticated user, team list, project list, or status list if `planning/linear.md` already provides the needed identifiers for this session.

## Workflow

1. Build the issue from the current chat context.
   - Use the discussion already in chat as the source material.
   - Ask brief follow-up questions only if the issue is still too vague to reach normal `Triaged` quality.
2. Draft a structured triage proposal for the user in this exact Markdown shape:

```markdown
# {Issue Title}

*{Issue Summary}*

### ❗ Problem Statement
{Problem Statement}

### ✨ Value Proposition
{Value Proposition}

### 📦 Scope
{Scope}

### 🛠️ Solution
{Proposed Solution}

### Priority
{Urgent | High | Medium | Low}

### 🧭 Priority Rationale
{Why this priority is appropriate}
```

3. Send that proposal in one visible `final` message and stop for user review.
4. If the user asks for refinements, revise only what they asked for and send one consolidated `final` message again.
5. Once the user approves the proposal, create a new Linear issue directly in `Triaged`.
6. Use this `Triaged` description template for the new issue:

```markdown
# {Issue Title}

## 🧾 Summary

Short summary of the issue and intended direction.

### ❗ Problem Statement

What is wrong, missing, confusing, or underperforming.

### ✨ Value Proposition

Why this is worth doing and what value it adds.

### 📦 Scope

What is in scope and any important non-goals or boundaries.

### 🛠️ Solution

Proposed direction that is clear enough for later planning, but not yet a detailed implementation plan.

### Priority

Urgent, High, Medium, or Low.

### 🧭 Priority Rationale

Why this priority is appropriate.
```

7. Apply exactly one kind label: `Feature`, `Improvement`, `Bug`, or `Admin`.
   - Use `Admin` when the primary expected work is administrative, provider-facing, dashboard-based, configuration-heavy, or otherwise external to the codebase, even if later code changes might still be needed.
8. Set the accepted Linear priority.
9. Confirm back to the user which issue was created.

## Promotion Standard

The new issue must already meet normal `Triaged` quality:

- clear title
- clear problem statement
- clear value proposition
- rough scope
- proposed solution that is only clear enough for later planning
- accepted priority with rationale
- exactly one kind label
- if the kind is `Admin`, the triage content makes it clear that the work is primarily operational or administrative rather than assumed code delivery

Do not use this path if the idea is still fuzzy enough that it really belongs in `Draft` or `Suggested`.

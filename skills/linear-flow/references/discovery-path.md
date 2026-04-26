# Discovery Path

Use this path for autonomous or semi-autonomous idea ingestion.

## When To Use

- Automation-driven competitor scouting
- Automation-driven product or feature scouting
- Codebase inspection for improvement opportunities
- User asks for idea discovery without asking to implement anything
- A user-approved non-blocking follow-up issue discovered during the Implementation Path that should be tracked later as a separate `Suggested` issue

## Scope

- Do not make code changes.
- Keep all output in chat and Linear only.
- Use `Suggested` for every issue created in this path.
- This is the default path for autonomous agent workflows started by automations.
- Create at most 10 new issues per discovery run.
- If `planning/linear.md` exists, use it first for team and project identifiers instead of rediscovering them with extra Linear calls.

## Workflow

1. Build context.
   - Understand the target product, project, or repository.
   - If the task involves external products, browse the web and collect evidence.
   - If the task involves internal improvement ideas, inspect the relevant codebase and existing product surface.

2. Check for duplicates.
   - Search the relevant Linear project or team for semantically similar issues before creating a new one.
   - Check at least `Suggested`, `Draft`, `Triaged`, `Todo` including its `Claimed` and `Ready` label sub-states, and `In Progress`.
   - If an existing issue already covers the same opportunity, do not create a duplicate. Reference the existing issue in the chat summary instead.

3. Create `Suggested` issues only.
   - If more than 10 candidate ideas survive duplicate filtering, keep only the strongest 10 and summarize the rest in chat.
   - Write a concise title.
   - Use this `Suggested` description template:

```markdown
# {Issue Title}

## 🧾 Summary

Brief description of the observed idea or opportunity.

## ✨ Why It May Matter

Why this could improve the product, workflow, or competitive position.

## 🔎 Evidence

- Link or observation 1
- Link or observation 2

## ➡️ Proposed Next Step

Review this idea and decide whether it should enter the backlog as a `Draft`.
```

   - Apply one kind label when confident enough to classify the idea: `Feature`, `Improvement`, `Bug`, or `Admin`.
   - Use `Admin` when the likely work is mainly dashboard, provider, configuration, policy, operational, or other external administrative work rather than repository implementation.

4. Report back.
   - Summarize what was added to Linear.
   - Highlight the strongest ideas and why they may deserve review.
   - Stop after documenting the ideas. Do not advance them beyond `Suggested`.

## Notes

- The user manually reviews and promotes `Suggested -> Draft`.
- Do not advance discovery-created issues beyond `Suggested`.

# Triage Path

Use this path to convert fuzzy backlog ideas into clearer `Triaged` issues.

## Scope

- Do not make code changes.
- Keep all work in chat and the Linear issue description.
- The goal is to shape ideas, not to produce a detailed implementation plan.
- This path is only for normal backlog ideas in `Draft`.
- `Assertion` issues are out of scope for this path and must live in `Suggested` for the dedicated `Assertion Path`.
- Keep the existing Linear issue title unchanged in the first triage pass.
- A title change is allowed only later in the same triage session if the user materially changes the scope or direction and the updated title would better reflect the approved triage result.
- Use `planning/linear.md` as the default source for team and project identifiers when that file exists.
- Do not spend Linear calls rediscovering the authenticated user, team list, project list, or status list if `planning/linear.md` already provides the needed identifiers for this triage session.

## Workflow

1. Fetch the first 7 issues from the `Draft` queue using the canonical queue-selection rule:
   - read `planning/linear.md` first and use its cached project id when available
   - use standard Linear MCP for normal issue reads and updates
   - use Composio raw GraphQL to query `Draft` issues with `first: 7`, `identifier`, `title`, and `sortOrder`
   - include `sort: [{ manual: { order: Ascending } }]` in the GraphQL query
   - treat the returned order as the real manual queue order
   - if this ordered query cannot be completed authoritatively, stop immediately and tell the user in `final`; do not continue triage with any fallback ordering
   - if an `Assertion`-labeled issue is encountered in `Draft`, stop immediately and tell the user in `final` that assertion issues belong in `Suggested` and must be handled through the `Assertion Path`
2. Read each issue and understand the intent.
3. Optionally inspect the codebase or product surface to confirm applicability and avoid misreading the request.
4. Prepare a structured triage proposal for the user in this exact Markdown shape:

```markdown
# {Issue Title}

{Issue ID}

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

---
```

5. Send one consolidated triage proposal message in `final` so it remains the visible user-facing deliverable.
6. After sending that message, stop. Do not send a follow-up message such as `waiting for your triage approval`, and do not make further tool calls until the user responds.
7. Treat the user's review reply as issue-by-issue input:
   - if the user says an issue is approved, mark it approved and do not show it again in later refinement rounds
   - if the user asks for changes on an issue, revise only that issue and keep it pending approval
   - if the user changes the issue scope materially, you may also propose a better title for that issue in the next revision round
8. If the user asks for refinements, send the revised proposal the same way, but include only the issues that still need approval. Use one consolidated `final` message, then stop again for the user's response.
9. Once every issue in the batch is approved, update each approved issue description with the generated text verbatim, using this `Triaged` description template:

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

10. If the user explicitly approved a title change for an issue, update the title at the same time as the description. Otherwise keep the original title unchanged.
11. Set the accepted Linear priority on each issue.
12. Move each approved issue to `Triaged`.
13. Ask whether the user wants to continue with the next 7 `Draft` issues using one visible `final` message, then stop for their response.

## Post-Triage Note

After an issue reaches `Triaged`, it stays there until the user manually promotes it to `Todo` for planning and implementation.

## Triage Proposal Message

Use the proposal shape above as the visible `final` message shown to the user.

This must be the last message before stopping for user review or refinement.

In refinement rounds, show only the issues that are still pending approval. Do not re-display issues the user already approved unless the user explicitly asks to revisit them.

## Promotion Gate

Only move `Draft -> Triaged` when all of the following are true:

- every issue being moved has been explicitly accepted by the user
- the description has been rewritten into the `Triaged` template
- the issue has exactly one kind label: `Feature`, `Improvement`, `Bug`, or `Admin`
- the user has accepted the proposed priority
- the Linear issue priority has been set to `Urgent`, `High`, `Medium`, or `Low`
- the proposed solution is clear enough for later planning but does not yet depend on unresolved detailed implementation decisions
- any title change has been explicitly approved by the user after a material scope change
- if the issue is labeled `Admin`, the triage result clearly communicates that the likely implementation path may be guidance-led, user-operated, externally configured, or mixed rather than purely code-driven

## Triage Standard

At this stage, produce:

- a clear problem statement
- a clear value proposition
- a rough scope
- a proposed solution that is only clear enough for later planning
- a proposed priority with rationale

If the issue is best classified as `Admin`, make that visible in the content itself:

- frame the problem and scope around the administrative or operational work to be done
- make clear whether code changes seem unnecessary, possible, or likely
- avoid forcing the issue into a code-first framing just because it is moving through the same Linear pipeline

Do not produce:

- detailed architecture
- exact schema changes
- rollout plans
- dependency decisions
- detailed testing plans

Those belong in the Planning Path.

## Priority Guidance

Use one of these four priorities during triage:

- `Urgent`: broken production behavior, security risk, blocked revenue path, or work that should interrupt the normal sequence
- `High`: strong product or business value, important blocker removal, or important near-term roadmap work
- `Medium`: worthwhile and planned, but not pressing
- `Low`: useful, but optional, polish-level, or safely deferrable

Do not use `No priority` in this workflow.

Priority helps clarify importance, but it does not override the user's manual ordering in Linear. The user may intentionally place a lower-priority issue above a higher-priority one for reasons such as implementation speed, batching, timing, or a tactical quick win.

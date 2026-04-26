# Planning Path

Use this path to turn the top unclaimed `Todo` issue into an implementation-ready specification without changing code.

## Scope

- Trigger this path only from a user chat session.
- Do not make code changes.
- Keep planning in the Linear issue description.
- Use chat to work through tradeoffs and unresolved decisions with the user.
- Think deeply, but keep the visible planning output concise.
- Scale the visible planning effort to the actual implementation complexity of the issue.
- This path claims the selected issue by adding the `Claimed` label from the `Planning` label group while leaving the issue in `Todo`.
- This path finalizes planning by replacing `Claimed` with `Ready` while leaving the issue in `Todo`.
- Use `planning/linear.md` as the default source for team and project identifiers when that file exists.
- Do not spend Linear calls rediscovering the authenticated user, team list, project list, or status list if `planning/linear.md` already provides the needed identifiers for this planning session.

## Workflow

1. Take the first unclaimed issue from the `Todo` queue using the canonical queue-selection rule:
   - read `planning/linear.md` first and use its cached project id, `Planning` label group id, and `Claimed` / `Ready` label ids when available
   - use standard Linear MCP for normal issue reads and updates
   - if the `Planning` label identifiers are missing or stale, fetch the team labels once, confirm the `Planning` group plus `Claimed` and `Ready` child labels, and update `planning/linear.md`
   - use Composio raw GraphQL to query `Todo` issues with `first: 1`, `identifier`, `title`, and `sortOrder`
   - filter out issues that already have any label under the `Planning` label group with `labels: { every: { parent: { id: { neq: $planningGroupId } } } }`
   - include `sort: [{ manual: { order: Ascending } }]` in the GraphQL query
   - treat the returned order as the real manual queue order
   - if this ordered query cannot be completed authoritatively, stop immediately and tell the user in `final`; do not continue planning with any fallback ordering
2. Add the `Claimed` label immediately so the issue is claimed for this planning session before any deeper work continues.
   - treat the GraphQL selection result and the `Claimed` label update as a critical claim sequence
   - as soon as the ordered query returns an issue id, add `Claimed` before doing anything else
   - do not read the issue description, inspect the repository, analyze the issue, send a progress update, or make any other tool call between receiving the selected issue id and adding `Claimed`
   - keep the issue in `Todo`
   - do not move it to `Planning`
   - do not remove any normal kind labels such as `Feature`, `Improvement`, `Bug`, or `Admin`
3. Read the issue description in full and distill it into a concise refresher for the user. Do not repeat the entire issue verbatim unless the user explicitly asks for that.
4. If the issue has gone stale, refresh it before planning further.
   - Treat an issue as stale when it has not been updated for 14 or more days, or when the relevant codebase area has materially changed since the last planning pass.
   - Revalidate the problem framing, assumptions, and implementation direction before continuing.
5. Explore the codebase deeply enough to understand:
   - how the current product behaves
   - how similar patterns are already implemented
   - which repository conventions should not be violated
6. Classify the issue into one planning depth before producing user-facing planning output:
   - `Fast`: localized, low-risk, straightforward work with little or no meaningful product, schema, auth, rollout, pricing, integration, or analytics ambiguity
   - `Standard`: moderate implementation work with some meaningful decisions, but still bounded enough that one concise planning pass should usually settle it
   - `Deep`: cross-cutting, high-risk, architecture-sensitive, or highly ambiguous work that needs fuller planning
7. State the chosen planning depth and one short reason in the first user-facing planning message.
8. Propose an implementation approach that covers only the materially relevant areas for this issue, such as:
   - affected user flows
   - architectural impact
   - data model impact
   - auth implications
   - analytics implications
   - observability implications
   - rollout or feature-flag implications
   - testing implications
   - commercial implications such as paywall, tiering, or add-ons when relevant
   - for `Admin` issues, whether the work is primarily user-operated external configuration, agent-guided operational work, repository changes, or a mixed path
9. Make explicit assumptions for low-risk details instead of asking about them.
10. Ask the user only about decisions whose answers would materially change implementation, such as architecture, schema, auth, rollout, pricing, integrations, major UX behavior, or validation strategy.
11. If the issue is `Fast` and no material decisions remain open:
   - skip Planning mode entirely
   - send the compact approval-gate message directly in `final`
   - stop for approval
12. If the issue is `Standard` and material decisions remain open:
   - ask the user to switch to Planning mode
   - use one concise Planning-mode handoff message in `final`
   - stop for the user's response
13. In `Standard` depth, ask one batch of 3 to 5 short multiple-choice questions by default. Only ask another batch if the plan is still blocked on material ambiguity.
14. If the issue is `Deep`:
   - ask the user to switch to Planning mode
   - use one concise Planning-mode handoff message in `final`
   - stop for the user's response
15. In `Deep` depth, ask questions in batches of 3 to 5 short multiple-choice prompts until the important decisions are settled. Do not generate a plan like you normally do when in plan mode.
16. Once the important decisions are settled, do not draft the plan yet. Instead, send one visible `final` message that tells the user the questioning phase is complete and asks them to switch you out of Planning mode before any plan text is generated.
17. That exit message must explicitly tell the user not to use the built-in `Implement this plan` control for this workflow, because this path is still in pre-implementation planning and must return to normal chat before continuing.
18. After sending that Planning-mode exit message, stop. Do not generate any plan artifact, implementation summary, or follow-up message while still in Planning mode, and do not make further tool calls until the user responds outside Planning mode.
19. After the user switches you out of Planning mode, state the other assumptions you made without asking and offer the user a chance to correct them.
20. Draft the proposed planning text in chat first using the compact chat templates below. Do not update the Linear issue description yet.
21. Before updating Linear or replacing `Claimed` with `Ready`, send one consolidated approval-gate message in `final` that includes the compact refresher, the planning depth, the finalized implementation-ready plan, the assumptions, the `Repo Planning Required: Yes/No` result, and the explicit approval request.
22. In revision rounds after the first approval-gate draft, show only the sections that changed unless the user explicitly asks to see the full plan again.
23. After sending that approval-gate message, stop. Do not send a follow-up message such as `waiting for planning approval`, and do not make further tool calls until the user responds.
24. Only after the user approves the proposed planning text may you rewrite the Linear issue description using the approved content.
25. When the user agrees the issue is fully specified and approved, remove the `Claimed` label and add the `Ready` label.
   - keep the issue in `Todo`
   - do not move it to any legacy planning status such as `Planning`, `Ready`, or `Prioritized`
26. If the planning session is abandoned after `Claimed` is added but before `Ready` is approved:
   - ask the user whether to release the claim or keep the issue claimed for later continuation
   - if the user approves releasing the claim, remove `Claimed` and leave the issue in `Todo`
   - do not add `Ready` unless the planning output was approved

## Conciseness And Revision Rules

- Think deeply before responding, but keep the visible planning output minimal.
- Prefer bullets over long prose when the same precision can be preserved.
- Do not restate already accepted sections unless they changed.
- Do not repeat the full issue description after the first concise refresher unless the user asks.
- In revision rounds, show only changed sections by default.
- Use the `Ready` description template for the final Linear write or when the user explicitly asks to see the full stored form. Use the compact chat templates for normal planning discussion.

## Planning Depth Rules

Choose exactly one depth:

- `Fast`: use for localized bug fixes, simple UI or copy adjustments, small behavior fixes, or other straightforward work that is unlikely to need planning-mode questions
- `Standard`: use for moderate product or codebase changes with a few material decisions
- `Deep`: use for new features, cross-cutting behavior, schema or auth changes, rollout-sensitive work, important integrations, or anything with broad implementation impact

If a supposedly `Fast` issue still has unresolved material decisions, promote it to `Standard` or `Deep` instead of forcing it through the fast path.

## Planning Output Standard

The planning description should be detailed enough that implementation can start with minimal ambiguity. Prefer the sections defined in the `Ready` template once the issue is finalized.

Always include `Repo Planning Required: Yes/No`.

For `Admin` issues, the planning output should still be implementation-ready, but it must not pretend the work is code-first when it is really operational or configuration-led. Make the operating mode explicit.

## Planning Mode Handoff Message

Use one visible `final` message to ask the user to switch to Planning mode. That message should include:

- the concise issue refresher, starting with a top-level `# {Issue Title}` heading
- the planning depth and a one-line reason
- the current proposed approach
- the current assumptions
- only the open material decisions
- the request to switch modes

This must be the last message before stopping for the user's response.

## Planning Mode Exit Message

When the question phase is complete, use one visible `final` message to tell the user:

- the important decisions are now settled enough to draft the plan
- they must switch you out of Planning mode before you continue
- they should not use the built-in `Implement this plan` control for this workflow
- you will draft the proposed planning text only after returning to normal chat

This must be the last message before stopping. Do not generate a plan artifact while still in Planning mode.

## Ready Approval Gate Message

Before replacing the `Claimed` label with `Ready`, send one visible `final` message that includes:

- the concise issue refresher, starting with a top-level `# {Issue Title}` heading
- the planning depth and reason
- the finalized implementation-ready plan
- the remaining assumptions
- the `Repo Planning Required: Yes/No` result
- the explicit approval request

This must be the last message before stopping for approval.

Do not replace the proposed plan with a note that Linear was updated. The user must be able to review the plan directly in chat before any Linear write happens.

In revisions after the first approval-gate draft, show only changed sections by default unless the user asks for the full plan again.

## Compact Chat Planning Template

Use this compact shape for user-facing planning discussion in chat:

```markdown
# {Issue Title}

Planning depth: {Fast | Standard | Deep}
Reason: {One short reason}

## 📋 Issue

{1 to 3 concise bullets or sentences}

## 🛠️ Proposed Approach

{Concise implementation approach}

## 📌 Assumptions

- Assumption 1
- Assumption 2
- Or `- None.`

## ❓ Open Questions / Decisions

- Open decision 1
- Open decision 2
- Or `- None.`

## 📝 Repo Planning Required

Yes or No.
```

## Fast Path Approval Template

Use this compact shape for `Fast` issues that do not need Planning mode:

```markdown
# {Issue Title}

Planning depth: Fast
Reason: {One short reason}

## 📋 Issue

{1 to 3 concise bullets or sentences}

## 🛠️ Proposed Approach

{Concise implementation approach}

## 📌 Assumptions

- Assumption 1
- Assumption 2
- Or `- None.`

## 📝 Repo Planning Required

Yes or No.

## ✅ Approval Request

Approve this concise plan and I will mark the issue ready.
```

## Ready Description Template

Use this once the issue is finalized and ready to receive the `Ready` label when writing the final approved content to Linear:

```markdown
# {Issue Title}

## 🧾 Summary

Short summary of the implementation-ready work.

## ❗ Problem

What problem this work solves.

## 🎯 Goals

- Goal 1
- Goal 2

## 🚫 Non-goals

- Non-goal 1
- Non-goal 2

## 🛠️ Final Implementation Plan

The agreed implementation approach.

## 🗃️ Data Model / Schema Impact

Final expected schema or data-model changes, or `None`.

## 🔌 Dependencies / Integrations

Final dependency and integration decisions.

## 📈 Analytics / Observability

Final analytics, logging, and monitoring requirements.

## 🚦 Rollout / Gating

Final rollout, feature-flag, paywall, or tiering decisions.

## 🧪 Validation / Testing

The required verification plan.

## 📝 Repo Planning Required

Yes or No.

## ✅ Acceptance Criteria

- Criterion 1
- Criterion 2

## ⛓️ Constraints

- Constraint 1
- Constraint 2
```

## Promotion Gate

Only add the `Claimed` label when the issue is the top unclaimed item in the `Todo` queue as determined by the canonical queue-selection rule and this planning session is intentionally starting on that issue.

Only replace `Claimed` with `Ready` when all of the following are true:

- the user has accepted the plan
- the user has approved the proposed planning text before it is written to Linear
- the issue description has been rewritten into the `Ready` template
- major product, technical, rollout, and validation decisions are settled
- any remaining uncertainty is minor enough not to block implementation
- `Repo Planning Required` is set explicitly to `Yes` or `No`
- for `Admin` issues, the implementation mode is clear enough that the later Implementation Path can tell whether it should guide external/admin execution, repository changes, or both

Do not move the issue out of `Todo` during this path.

If planning is abandoned before `Ready` approval, do not leave a stale `Claimed` label silently. Ask the user whether to release the claim, then remove only `Claimed` if they approve.

## Repo Planning Rule Check

Before finalizing the plan, compare the repository's planning rules with this Linear-only planning workflow.

The common discrepancy is:

- This path keeps planning in Linear until implementation begins.
- Some repositories require file-based planning artifacts for major changes before implementation.

Treat that as follows:

- For small or isolated features, Linear-only planning is usually consistent with repo rules.
- For major architecture, data model, routing, permissions, or integration changes, tell the user that repository planning docs will need to be created during the Implementation Path before code changes begin.

Do not create those repo planning files yet in this path. Only surface the requirement clearly.

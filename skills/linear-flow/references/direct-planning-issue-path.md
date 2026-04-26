# Direct Planning Issue Path

Use this path when the user already discussed an issue in chat and wants a new Linear issue created directly into the planning workflow.

## Scope

- Do not make code changes.
- Keep all work in chat and the Linear issue description.
- Do not take an issue from the `Todo` queue in this path.
- Use this path only when the current chat context is already beyond normal `Triaged` quality and is ready to enter planning immediately.
- If the issue is still only triage-level clear, use the `Direct Triaged Issue Path` instead.
- Use `planning/linear.md` as the default source for team and project identifiers when that file exists.
- Do not spend Linear calls rediscovering the authenticated user, team list, project list, or status list if `planning/linear.md` already provides the needed identifiers for this session.

## Workflow

1. Build the issue from the current chat context.
   - Use the ongoing discussion, feedback source, and any linked evidence already in chat as the source material.
   - This commonly includes Sentry, PostHog, Google Ads, GA4, or other feedback-driven context, but the path is not limited to those sources.
2. Decide whether the issue is actually ready for direct planning.
   - If it is still fuzzy enough that it mainly needs problem/value/scope shaping, stop and route to the `Direct Triaged Issue Path` instead.
   - If it is already concrete enough to begin planning immediately, continue.
3. Create the new Linear issue directly in `Todo` with the `Claimed` label from the `Planning` label group.
4. Then run the same planning-depth workflow used by the normal `Planning Path`:
   - classify the issue as `Fast`, `Standard`, or `Deep`
   - keep user-facing output concise
   - ask only about decisions that materially affect implementation
   - use Planning mode only when the chosen depth actually requires it
   - show only changed sections in revision rounds by default
5. Before creating the issue in Linear, send one visible `final` message that includes:
   - the proposed title
   - the concise issue refresher
   - why this qualifies for direct planning instead of direct triage
   - the initial planning depth and reason
   - the current proposed approach
   - the current assumptions
   - the explicit approval request
6. After sending that message, stop. Do not send a follow-up waiting message, and do not create the Linear issue until the user approves.
7. Once the user approves:
   - create the new issue in `Todo`
   - add the `Claimed` label immediately
   - continue through the planning workflow exactly as the normal `Planning Path` would from that point onward
8. Replace `Claimed` with `Ready` only when the planning work is fully approved. Keep the issue in `Todo`.

## Approval Gate Message

Use this exact shape for the first approval-gate message in `final`:

```markdown
# {Issue Title}

## 📋 Issue

{1 to 3 concise bullets or sentences}

## 🛠️ Planning Entry

This issue is ready to enter the planning workflow directly from the current discussion.

## 🛠️ Proposed Approach

{Concise implementation approach}

## 📌 Assumptions

- Assumption 1
- Assumption 2
- Or `- None.`

## 📝 Initial Planning Depth

{Fast | Standard | Deep} — {One short reason}

## ✅ Approval Request

Approve creating this issue directly in `Todo` with the `Claimed` label and continuing the planning workflow from there.
```

## Promotion Standard

Only use this path when all of the following are true:

- the chat context already contains enough substance to justify planning immediately
- the issue is more mature than a normal `Triaged` intake
- the agent can already name a plausible implementation direction
- the remaining unknowns are planning-level decisions, not basic triage framing

If those conditions are not met, use the `Direct Triaged Issue Path` instead.

## Relationship To Planning Path

After the issue is created in `Todo` with the `Claimed` label, follow the same planning-depth, conciseness, assumption, Planning-mode, and `Ready` label approval rules defined in [planning-path.md](planning-path.md).

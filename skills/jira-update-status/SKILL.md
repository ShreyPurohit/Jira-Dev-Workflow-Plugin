---
name: jira-update-status
description: Transition a Jira issue to an arbitrary status via dynamic discovery. Use for general status changes EXCEPT the two specialized cases — starting work (To Do → In Progress, use jira-start-work) and final completion (comment + done, use jira-complete).
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Update Status

Transition a Jira issue between statuses safely using dynamic transition discovery.
This is the **general-purpose** transition skill for any status change that is not
one of the two specialized flows.

## When to use

- User says "move PROJ-123 to [status]"
- User says "transition PROJ-123 to [status]"
- User says "mark PROJ-123 as [status]"
- User says "update the status of PROJ-123"
- User says "reopen PROJ-123" or "move PROJ-123 back to [status]"

## When NOT to use

- **Moving To Do → In Progress to begin work** → use `jira-start-work`.
- **Final completion** (adds a completion comment + transitions to done/sign-off)
  → use `jira-complete`.
- **Only posting a comment, no status change** → use `jira-comment`.

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Call primary tools directly: `getJiraIssue`, `transitionJiraIssue`.
4. Listing transitions is deferred: `discover` → `executeRead` with `listJiraIssueTransitions`.

## How to respond

### Step 1: Read current state

1. Call `getJiraIssue` with `cloudId` to confirm the current status.
2. Report: "PROJ-123 is currently in [current status]."

### Step 2: Discover transitions

1. Use `discover` to identify the `listJiraIssueTransitions` operation and its schema.
2. Use `executeRead` with the discovered operation to retrieve the available transitions.
3. Parse the available transitions. Match on the **destination status**, not the transition label. Typical fields:
   - `id` — the transition ID to execute
   - `name` — the transition's label (DO NOT match on this)
   - destination status — often `to.name`, or an equivalent status name the tool returns (MATCH ON THIS)

If the payload does not use `to.name`, use the field that names where the issue will land after the transition. Never invent a destination.

### Step 3: Find the correct transition

**CRITICAL RULE: Always match by destination status, never by transition label (`name`).**

Transition names are arbitrary workflow labels:

- A transition named "Transfer" might lead to "READY FOR SIGN OFF"
- A transition named "Start Progress" might lead to "In Progress"
- A transition named "Done" might lead to "Closed"

The user speaks in terms of destination statuses ("move to In Progress"), so match their intent against the destination status field in the tool response.

### Step 4: Confirm and execute

1. **Always confirm** before executing: "I'll transition PROJ-123 from [current] to [destination]. The transition is '[transition name]' (ID: [id]). Proceed?"
2. On confirmation, call `transitionJiraIssue` with the transition ID and `cloudId`.
3. Verify by re-reading the issue: confirm the status actually changed.

### Step 5: Report

- ✅ "PROJ-123 transitioned from [old] to [new]"
- Or ❌ "Transition failed: [reason]"

## Important rules

- **NEVER match transitions by label.** The `name` field is an arbitrary workflow label. Only the destination status tells you where the transition leads.
- **Always confirm before transitioning.** This is a write operation.
- **Present available options** when the requested destination isn't available: "PROJ-123 cannot move to [requested]. Available destinations from [current]: [list of destination status names]."
- **Verify after transition.** Re-read the issue to confirm the status changed.
- **Case-insensitive matching.** The user might say "in progress" while Jira has "In Progress". Match case-insensitively.
- **Partial matching.** "ready for sign off" should match "READY FOR SIGN OFF". Use case-insensitive substring matching.

## Common workflow transitions

These are EXAMPLES — always discover the transition operation dynamically via
`discover`, then use `executeRead` to retrieve transitions.

| User says          | Likely destination status      |
| ------------------ | ------------------------------ |
| "start work"       | In Progress                    |
| "mark as done"     | Done                           |
| "ready for review" | READY FOR SIGN OFF / In Review |
| "reopen"           | To Do / Open / Reopened        |
| "close"            | Done / Closed                  |

## Error handling

- **No matching transition**: "PROJ-123 is in [status] and cannot move to [requested]. Available transitions lead to: [list]. Which would you like?"
- **Transition requires fields**: "This transition requires additional fields: [list]. Please provide: [details]."
- **Permission denied**: "You don't have permission to perform this transition on PROJ-123."

## Example

User: "Move PROJ-123 to In Review"

Response:

1. Fetch PROJ-123 → Status: "In Progress"
2. Get transitions via `discover` + `executeRead` (`listJiraIssueTransitions`) → match the destination status (example shape: `{id: "31", name: "Submit for Review", to: {name: "In Review"}}`)
3. Confirm: "I'll transition PROJ-123 from In Progress → In Review via the 'Submit for Review' transition. Proceed?"
4. On yes: Execute `transitionJiraIssue` with the discovered transition ID and `cloudId`
5. Verify: Re-read issue, confirm status is "In Review"
6. Report: "✅ PROJ-123 is now In Review"

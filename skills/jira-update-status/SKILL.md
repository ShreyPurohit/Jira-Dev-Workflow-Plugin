---
name: jira-update-status
description: Transition Jira issues between statuses using dynamic discovery
---

# Jira Update Status

Transition a Jira issue between statuses safely using dynamic transition discovery.

## When to use

- User says "move PROJ-123 to [status]"
- User says "transition PROJ-123 to [status]"
- User says "mark PROJ-123 as [status]"
- User says "update the status of PROJ-123"
- User says "close PROJ-123" or "reopen PROJ-123"

## How to respond

### Step 1: Read current state

1. Call `jira_get_issue` to confirm the current status.
2. Report: "PROJ-123 is currently in [current status]."

### Step 2: Discover transitions

1. Call `jira_get_transitions` for the issue.
2. Parse the available transitions. Each transition has:
   - `id` — the transition ID to execute
   - `name` — the transition's label (DO NOT match on this)
   - `to.name` — the DESTINATION status name (MATCH ON THIS)

### Step 3: Find the correct transition

**CRITICAL RULE: Always match by `to.name` (destination status), never by `name` (transition label).**

Transition names are arbitrary workflow labels:

- A transition named "Transfer" might lead to "READY FOR SIGN OFF"
- A transition named "Start Progress" might lead to "In Progress"
- A transition named "Done" might lead to "Closed"

The user speaks in terms of destination statuses ("move to In Progress"), so match their intent against `to.name`.

### Step 4: Confirm and execute

1. **Always confirm** before executing: "I'll transition PROJ-123 from [current] to [destination]. The transition is '[transition name]' (ID: [id]). Proceed?"
2. On confirmation, call `jira_transition_issue` with the transition ID.
3. Verify by re-reading the issue: confirm the status actually changed.

### Step 5: Report

- ✅ "PROJ-123 transitioned from [old] to [new]"
- Or ❌ "Transition failed: [reason]"

## Important rules

- **NEVER match transitions by name.** The `name` field is an arbitrary label. Only `to.name` tells you where the transition leads.
- **Always confirm before transitioning.** This is a write operation.
- **Present available options** when the requested destination isn't available: "PROJ-123 cannot move to [requested]. Available destinations from [current]: [list of to.name values]."
- **Verify after transition.** Re-read the issue to confirm the status changed.
- **Case-insensitive matching.** The user might say "in progress" while Jira has "In Progress". Match case-insensitively.
- **Partial matching.** "ready for sign off" should match "READY FOR SIGN OFF". Use case-insensitive substring matching.

## Common workflow transitions

These are EXAMPLES — always discover dynamically via `jira_get_transitions`:

| User says          | Likely destination (`to.name`) |
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
2. Get transitions → Found: `{id: "31", name: "Submit for Review", to: {name: "In Review"}}`
3. Confirm: "I'll transition PROJ-123 from In Progress → In Review via the 'Submit for Review' transition. Proceed?"
4. On yes: Execute `jira_transition_issue(issue_key="PROJ-123", transition_id="31")`
5. Verify: Re-read issue, confirm status is "In Review"
6. Report: "✅ PROJ-123 is now In Review"

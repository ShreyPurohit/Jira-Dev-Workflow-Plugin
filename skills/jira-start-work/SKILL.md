---
name: jira-start-work
description: Move a Jira issue into an in-progress state safely and verify the result
---

# Jira Start Work

Begin development on a Jira issue by moving it to the appropriate in-progress state.

## When to use

- User says "start work on PROJ-123"
- User says "begin PROJ-123"
- User says "pick up PROJ-123"
- User says "I'm starting PROJ-123"
- User wants to transition a ticket to In Progress

## How to respond

### Step 1: Verify the ticket

1. Call `jira_get_issue` to read the current state.
2. Verify:
   - The ticket exists
   - The ticket is in a state that can be transitioned to "In Progress" (typically "To Do" or equivalent)
   - Note the current status

### Step 2: Transition to In Progress

1. Call `jira_get_transitions` to discover available transitions.
2. Find the transition whose **destination status** (`to.name`) matches "In Progress" (or the user's target status).
   - **CRITICAL**: Match by `transition.to.name` (destination), NOT by `transition.name` (the transition label). Transition names are arbitrary workflow labels that rarely match the destination status.
3. Confirm with the user: "I'll transition PROJ-123 from [current] to In Progress. Proceed?"
4. On confirmation, call `jira_transition_issue` with the transition ID.
5. Verify the transition succeeded by re-reading the issue.

### Step 3: Summarize

Report:

- ✅ Ticket transitioned to In Progress
- 📋 Key acceptance criteria to keep in mind

## Important rules

- **Always confirm before transitioning.** Never silently change ticket status.
- **Match transitions by destination status** (`to.name`), not by transition name.
- **If no valid transition exists**, report it clearly: "PROJ-123 is currently in [status] and has no available transition to In Progress. Available transitions: [list destinations]."
- **Do not assign the ticket** unless the user explicitly asks.
- **Do not add comments** during start-work — that's a separate action.
- **Do not create a Git branch** as part of starting work.
- **Do not suggest or inspect branch names, checkout commands, or Git state** during start-work.

## Error handling

- **No transition to In Progress available**: List available transitions with their destinations so the user can choose.
- **Ticket already In Progress**: "PROJ-123 is already In Progress. No status change is needed."
- **Ticket is Done**: "PROJ-123 is already Done. Do you want to reopen it?"

## Example

User: "Start work on PROJ-456"

Response:

1. Fetch PROJ-456 → Status: "To Do"
2. Get transitions → Find transition with `to.name = "In Progress"`
3. Confirm: "I'll move PROJ-456 from To Do → In Progress. Proceed?"
4. On yes: Execute transition, verify
5. Report the resulting status without creating or suggesting a branch

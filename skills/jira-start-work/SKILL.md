---
name: jira-start-work
description: Begin work on a Jira issue by transitioning it to In Progress. Use only for starting work; for any other status change use jira-update-status, and use jira-branch separately to create a Git branch.
---

# Jira Start Work

Begin development on a Jira issue by moving it to the appropriate In Progress state.
This is the **specialized** transition for starting work (fixed destination: In
Progress) — a narrowed case of the general `jira-update-status`.

## When to use

- User says "start work on PROJ-123"
- User says "begin PROJ-123"
- User says "pick up PROJ-123"
- User says "I'm starting PROJ-123"

## When NOT to use

- **Transitioning to any status other than In Progress** → use `jira-update-status`.
- **Creating the Git branch** → use `jira-branch` separately.
- **Final completion** → use `jira-complete`.

## How to respond

### Step 1: Verify the ticket

1. Call `getJiraIssue` to read the current state.
2. Verify:
   - The ticket exists
   - The ticket is in a state that can be transitioned to "In Progress" (typically "To Do" or equivalent)
   - Note the current status

### Step 2: Transition to In Progress

Use the transition discovery, destination matching, confirmation, execution, and
verification procedure defined by `jira-update-status`. That procedure uses
`discover` to identify `listJiraIssueTransitions`, `executeRead` to retrieve
transitions, and `transitionJiraIssue` to perform the confirmed write.

- The target is the workflow's **In Progress equivalent**.
- Match the destination using `transition.to.name`, never the transition label
  in `transition.name`.
- Always confirm before changing Jira state.
- Verify the resulting status afterward.

### Step 3: Summarize

Report:

- ✅ Ticket transitioned to In Progress
- 📋 Key acceptance criteria to keep in mind

## Important rules

- **Always confirm before transitioning.** Never silently change ticket status.
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
2. Apply the `jira-update-status` transition procedure for the workflow's In Progress equivalent
3. Report the resulting status without creating or suggesting a branch

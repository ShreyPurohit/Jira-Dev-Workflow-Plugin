---
name: jira-complete
description: Wrap up a finished Jira issue — post a confirmed completion comment AND transition it to the final Done status in one flow. Use only for final completion; for a bare status change use jira-update-status.
---

# Jira Complete

Complete a development workflow on a Jira issue: verify work is done, prepare
and confirm a completion comment, then transition to the final Done status.

## Responsibility

This skill owns completion intent and coordinates the completion workflow. It
uses `jira-update-status` for generic transition discovery, destination matching,
confirmation, execution, and verification. It uses `jira-comment` for generic
comment creation and confirmation rules.

It does not own generic status-transition mechanics or generic Jira comment
creation, and it does not create branches or link Git work to Jira.

## When to use

- User says "complete PROJ-123"
- User says "finish PROJ-123 and mark it done"
- User says "wrap up PROJ-123"
- User says "PROJ-123 is done — close it out with a summary"

## When NOT to use

- **Just changing the status, no completion comment** → use `jira-update-status`
  (this skill always pairs a comment with the transition).
- **Ready for review or ready for sign-off, but not final completion** → use
  `jira-update-status`.
- **Posting a comment without transitioning** → use `jira-comment`.
- **Creating or linking Git work** → use `jira-branch` or `jira-link-work`.

## How to respond

### Step 1: Verify current state

1. Call `getJiraIssue` to read the current status.
2. Confirm it's in a state that makes sense to complete (typically "In Progress").
3. If it's already Done, inform the user.

### Step 2: Determine the target status

Use the final Done-equivalent status required by the user's completion request.
If the request does not identify a final completion destination, ask the user
before proceeding. Intermediate review or sign-off statuses belong to
`jira-update-status`.

### Step 3: Prepare the completion comment

Prepare a completion comment from information explicitly supplied by the user or
actually gathered through the appropriate capabilities. Do not invent test,
build, branch, commit, pull request, file-change, or acceptance-criteria
results.

```markdown
## Development Complete

### Summary of changes

- [key changes]

### Acceptance criteria status

- [x] [criteria explicitly verified from available information]
```

### Step 4: Confirm and coordinate execution

1. Show the complete proposed completion comment and confirm it with the user
   before posting, using `jira-comment` for comment creation.
2. Pass the requested final destination to `jira-update-status`, which owns
   transition discovery, matching by `to.name`, confirmation, execution, and
   verification.
3. Coordinate the two operations and report whether the comment and transition
   each succeeded. If the comment fails but the transition succeeds, report
   partial success and offer to retry the comment.

### Step 5: Report

```
✅ PROJ-123 workflow complete:
- Status: In Progress → Done
- Comment: Added completion summary
```

## Important rules

- **Coordinate the lower-level skills.** Use `jira-update-status` for transition mechanics and `jira-comment` for comment mechanics.
- **Don't auto-complete without user intent.** Never transition a ticket just because code was committed — the user decides when a ticket is done.
- **Do not report Git or verification details unless they were actually gathered or explicitly supplied.**

## Workflow patterns

### Simple workflow (To Do → In Progress → Done)

- "Complete PROJ-123" → transition to Done

### Final completion after review

- Intermediate review or sign-off transitions are handled by `jira-update-status`.
- "Complete PROJ-123" → prepare and confirm a completion comment, then transition to the discovered final Done-equivalent status.

## Error handling

- **No available transition to target**: List available destinations and ask which the user wants.
- **Ticket already in target status**: "PROJ-123 is already in [status]. Nothing to do."
- **Comment fails but transition succeeds**: Report partial success and offer to retry the comment.

## Example

User: "PROJ-123 is complete — all ACs pass, tests green. Mark it done."

Response:

1. Fetch PROJ-123 → Status: "In Progress"
2. Determine the final Done-equivalent destination from the user's intent
3. Prepare and show the completion comment, then confirm before posting
4. Coordinate comment posting through `jira-comment` and the status change through `jira-update-status`
5. Report only the comment and transition results actually verified

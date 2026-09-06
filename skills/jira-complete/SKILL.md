---
name: jira-complete
description: Wrap up a finished Jira issue — post a completion comment AND transition it to the done/sign-off status in one flow. Use only for final completion; for a bare status change use jira-update-status.
---

# Jira Complete

Complete a development workflow on a Jira issue: verify work is done, add a
completion comment, and transition to the done/sign-off status — the two actions
together, as one wrap-up flow.

## Responsibility

This skill owns completion intent and coordinates the completion workflow. It
uses `jira-update-status` for generic transition discovery, destination matching,
confirmation, execution, and verification. It uses `jira-comment` for generic
comment creation and confirmation rules when a completion comment is needed.

It does not own generic status-transition mechanics or generic Jira comment
creation, and it does not create branches or link Git work to Jira.

## When to use

- User says "complete PROJ-123"
- User says "finish PROJ-123 and mark it done"
- User says "wrap up PROJ-123"
- User says "PROJ-123 is done — close it out with a summary"
- User says "PROJ-123 is ready for sign off"

## When NOT to use

- **Just changing the status, no completion comment** → use `jira-update-status`
  (this skill always pairs a comment with the transition).
- **Preparing a PR description / moving to review** (not final done) → use the
  appropriate review workflow outside this completion skill.
- **Posting a comment without transitioning** → use `jira-comment`.

## How to respond

### Step 1: Verify current state

1. Call `jira_get_issue` to read the current status.
2. Confirm it's in a state that makes sense to complete (typically "In Progress").
3. If it's already Done, inform the user.

### Step 2: Determine the target status

Ask or infer from context:

- "Ready for review" / "ready for sign off" → transition to review status
- "Done" / "close" → transition to Done
- "Complete" → depends on workflow; ask if ambiguous

### Step 3: Optionally add a completion comment

If the user has provided context about what was done, offer to add a completion comment:

```markdown
## Development Complete

**Branch:** feat/PROJ-123-description
**Build:** ✅ Passing
**Tests:** ✅ [count] tests passing

### Summary of changes

- [key changes]

### Acceptance criteria status

- [x] AC-1: [met]
- [x] AC-2: [met]
```

### Step 4: Coordinate execution

1. If a completion comment is needed, pass it through `jira-comment`'s comment
   creation and confirmation rules.
2. Pass the requested destination to `jira-update-status`, which owns transition
   discovery, matching by `to.name`, confirmation, execution, and verification.
3. Coordinate the two operations and report whether the comment and transition
   each succeeded. If the comment fails but the transition succeeds, report
   partial success and offer to retry the comment.

### Step 5: Report

```
✅ PROJ-123 workflow complete:
- Status: In Progress → READY FOR SIGN OFF
- Comment: Added completion summary
- Branch: feat/PROJ-123-description
```

## Important rules

- **Coordinate the lower-level skills.** Use `jira-update-status` for transition mechanics and `jira-comment` for comment mechanics.
- **Don't auto-complete without user intent.** Never transition a ticket just because code was committed — the user decides when a ticket is done.
- **If QA is part of the workflow**, clarify whether "complete" means "ready for QA" or "fully done". Don't skip workflow stages.

## Workflow patterns

### Simple workflow (To Do → In Progress → Done)

- "Complete PROJ-123" → transition to Done

### Review workflow (To Do → In Progress → Ready for Review → Done)

- "PROJ-123 is ready for review" → transition to Ready for Review
- "Mark PROJ-123 as done" → transition to Done

### Sign-off workflow (To Do → In Progress → READY FOR SIGN OFF → Done)

- "PROJ-123 is ready for sign off" → transition to READY FOR SIGN OFF
- Final "Done" transition happens after human sign-off

## Error handling

- **No available transition to target**: List available destinations and ask which the user wants.
- **Ticket already in target status**: "PROJ-123 is already in [status]. Nothing to do."
- **Comment fails but transition succeeds**: Report partial success and offer to retry the comment.

## Example

User: "PROJ-123 is complete — all ACs pass, tests green. Mark it done."

Response:

1. Fetch PROJ-123 → Status: "In Progress"
2. Determine the completion destination from the user's intent
3. Coordinate the completion comment through `jira-comment` and the status change through `jira-update-status`
4. Report: "✅ PROJ-123 moved to Done with completion summary"

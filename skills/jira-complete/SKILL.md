---
name: jira-complete
description: Wrap up a finished Jira issue — post a completion comment AND transition it to the done/sign-off status in one flow. Use only for final completion; for a bare status change use jira-update-status.
---

# Jira Complete

Complete a development workflow on a Jira issue: verify work is done, add a
completion comment, and transition to the done/sign-off status — the two actions
together, as one wrap-up flow.

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

### Step 4: Execute the transition

1. Call `jira_get_transitions` to discover available transitions.
2. Match by **destination status** (`to.name`), not by transition name.
3. Confirm: "I'll transition PROJ-123 from [current] to [destination] and add a completion comment. Proceed?"
4. On confirmation:
   - Add the comment (if applicable)
   - Execute the transition
   - Verify success

### Step 5: Report

```
✅ PROJ-123 workflow complete:
- Status: In Progress → READY FOR SIGN OFF
- Comment: Added completion summary
- Branch: feat/PROJ-123-description
```

## Important rules

- **Always confirm before transitioning and commenting.** This is a combined write operation.
- **Match transitions by destination** (`to.name`), never by transition name.
- **Don't auto-complete without user intent.** Never transition a ticket just because code was committed — the user decides when a ticket is done.
- **Verify the transition succeeded.** Re-read the issue to confirm.
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
2. Get transitions → Find transition to "Done" or "In Review"
3. Confirm: "I'll add a completion comment and transition PROJ-123 to Done. Proceed?"
4. On yes:
   - Add structured comment with AC status and branch info
   - Execute transition via the discovered transition ID
   - Verify status changed
5. Report: "✅ PROJ-123 moved to Done with completion summary"

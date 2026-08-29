---
name: jira-comment
description: Add structured progress or completion comments to Jira issues
---

# Jira Comment

Add structured progress or completion comments to Jira issues.

## When to use

- User says "add a comment to PROJ-123"
- User says "update PROJ-123 with progress"
- User says "post implementation notes to PROJ-123"
- User says "document what was done on PROJ-123"
- User wants to record development progress on a ticket

## How to respond

### Adding a comment

1. Determine the comment content:
   - If the user provides specific text, use that.
   - If the user asks to document progress, construct a structured comment.

2. **Always confirm** the comment content before posting: "I'll add this comment to PROJ-123:\n\n[comment preview]\n\nProceed?"

3. On confirmation, call `jira_add_comment` with the issue key and formatted body.

4. Confirm success: "✅ Comment added to PROJ-123."

### Structured comment templates

#### Progress update

```markdown
## Development Progress

**Branch:** feat/PROJ-123-description
**Status:** In Progress

### Completed

- [what's been done]

### In Progress

- [current work]

### Remaining

- [what's left]
```

#### Implementation complete

```markdown
## Implementation Complete

**Branch:** feat/PROJ-123-description
**Files changed:** [count]

### Changes

- [summary of changes]

### Testing

- [tests added/run]

### Ready for review

[any notes for reviewer]
```

#### Bug fix

```markdown
## Bug Fix

**Root cause:** [brief explanation]
**Fix:** [what was changed]
**Branch:** fix/PROJ-123-description

### Verification

- [how it was verified]
```

## Important rules

- **Always confirm before posting.** Comments are visible to the whole team.
- **Use markdown formatting.** Jira Cloud renders markdown in comments.
- **Keep comments concise and actionable.** Don't dump entire file diffs — summarize what changed and why.
- **Don't add comments for trivial status updates** that the transition itself communicates. If the user just transitioned to "In Progress", a comment saying "Started work" adds no value.
- **Include the branch name** when documenting development work — it links the comment to the code.
- **Never include secrets, tokens, or sensitive paths** in comments.

## Error handling

- **Issue not found**: "I couldn't find PROJ-123. Please verify the issue key."
- **Permission denied**: "You don't have permission to comment on PROJ-123."
- **Empty comment**: "What would you like the comment to say?"

## Example

User: "Add a progress comment to PROJ-123 — I've finished the login redesign and tests pass"

Response:

1. Construct comment:

   ```
   ## Development Progress

   **Branch:** feat/PROJ-123-login-redesign
   **Status:** Implementation complete

   ### Completed
   - Login page redesign with responsive layout
   - Unit tests passing

   ### Ready for
   - Code review
   - QA validation
   ```

2. Confirm with user
3. Post via `jira_add_comment`
4. Report success

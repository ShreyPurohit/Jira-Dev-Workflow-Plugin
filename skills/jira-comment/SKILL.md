---
name: jira-comment
description: Post a free-text progress, note, or completion comment on a Jira issue. Use for prose updates the user dictates or asks you to compose — NOT for linking git branches/commits/PRs (use jira-link-work) and NOT for status changes (use jira-update-status).
---

# Jira Comment

Post a **free-text** comment (progress note, clarification, or completion summary)
on a Jira issue. This skill owns prose comments the user dictates or asks you to
compose.

## When to use

- User says "add a comment to PROJ-123"
- User says "comment on PROJ-123 that ..."
- User says "post implementation notes to PROJ-123"
- User says "leave a note on PROJ-123"
- User dictates comment text they want posted verbatim

## When NOT to use

- **Linking a git branch, commit, or PR to the issue** → use `jira-link-work`
  (it builds the comment _from git data_; this skill is for free-text prose).
- **Changing the issue's status** → use `jira-update-status` (or `jira-complete`
  for a wrap-up comment + transition together).
- **Reading or summarizing existing comments** → use `jira-read`.

## How to respond

### Adding a comment

1. Determine the comment content:

- If the user provides exact comment text and explicitly asks to post it, use
  that request as confirmation.
- If the user asks to document progress, construct a structured comment.

2. If the comment is generated, reformatted, or substantially drafted by the
   skill, show the proposed comment and ask for confirmation before posting:
   "I'll add this comment to PROJ-123:\n\n[comment preview]\n\nProceed?"

3. On confirmation, call `addOrEditJiraIssueComment` with the issue key and formatted body.

4. Confirm success: "✅ Comment added to PROJ-123."

### Structured comment templates

#### Progress update

```markdown
## Development Progress

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

### Verification

- [how it was verified]
```

## Important rules

- **Confirm generated or substantially drafted comments before posting.** Exact comment text explicitly provided by the user for posting does not require a second confirmation.
- **Use markdown formatting.** Jira Cloud renders markdown in comments.
- **Keep comments concise and actionable.** Don't dump entire file diffs — summarize what changed and why.
- **Don't add comments for trivial status updates** that the transition itself communicates. If the user just transitioned to "In Progress", a comment saying "Started work" adds no value.
- **Do not inspect Git state or require Git context** for a generic comment. A branch, commit, or PR may be included only when the user explicitly provides it or explicitly asks for Git/development context. For Git-derived comments, use `jira-link-work`.
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

   **Status:** Implementation complete

   ### Completed
   - Login page redesign with responsive layout
   - Unit tests passing

   ### Ready for
   - Code review
   - QA validation
   ```

2. Confirm with user
3. Post via `addOrEditJiraIssueComment`
4. Report success

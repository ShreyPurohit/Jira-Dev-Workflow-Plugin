---
name: jira-branch
description: Create a Git branch from a Jira issue key with smart naming and repository safety checks
---

# Jira Branch

Create a git feature branch from a Jira issue, with a smart slug derived from the issue summary.

## When to use

- User says "create a branch for PROJ-123"
- User says "start a branch for PROJ-456"
- User says "branch PROJ-789"
- User says "set up a branch for this ticket"
- User mentions an issue key in the context of starting code work

## How to respond

### Step 1: Fetch the issue

1. Call `jira_get_issue` with the issue key to get the summary and status.
2. If the issue doesn't exist, report the error and stop.

### Step 2: Generate branch name

Create a branch name following the pattern:

```
feat/<ISSUE-KEY>-<slug>
```

Rules for the slug:
- Derive from the issue summary
- Lowercase only
- Replace spaces and special characters with hyphens
- Remove consecutive hyphens
- Trim to max 50 characters (excluding the `feat/<KEY>-` prefix)
- Remove trailing hyphens

**Examples:**
- `PROJ-123` "Add email verification to user settings" → `feat/PROJ-123-add-email-verification-to-user-settings`
- `PROJ-42` "Fix login page CSS on mobile devices" → `feat/PROJ-42-fix-login-page-css-on-mobile-devices`
- `BUG-101` "[Critical] API timeout on /users endpoint under load" → `feat/BUG-101-api-timeout-on-users-endpoint-under-load`

### Step 3: Check repository state

Before creating the branch:
1. Inspect the current Git repository state.
2. Check whether the branch already exists locally.
3. Warn the user about uncommitted or conflicting work before creating or switching branches.
4. If the working tree is dirty, ask for confirmation before continuing.

### Step 4: Confirm with user

Present the proposed branch name and ask for confirmation before creating:

```
Issue: PROJ-123 — Fix login page CSS
Status: To Do
Proposed branch: feat/PROJ-123-fix-login-page-css

Create this branch from the current branch?
```

If the user wants a different name, use their preference.

### Step 5: Create the branch

After confirmation, execute:
1. `git checkout -b <branch-name>`
2. Verify the branch was created successfully by checking the active branch name.

## Rules

- **Never create a branch without user confirmation** of the branch name.
- **Never force-checkout** — if there are uncommitted changes, warn the user first.
- **Always use `feat/` prefix** unless the user specifies otherwise (e.g., `fix/`, `chore/`).
- **Branch slug comes from the issue summary** — never invent or guess content.
- **Do not transition the Jira issue to In Progress** as part of branch creation.
- **Read Jira issue data only to determine the issue key and branch naming context** — status mutation is outside this skill's responsibility.

## Error handling

- **Issue not found**: Report the error clearly with the key that failed.
- **Branch already exists**: Inform the user and ask if they want to switch to it instead.
- **Dirty working tree**: Warn about uncommitted changes before any checkout.
- **Repository not in a safe state**: Report what is blocking the branch creation and let the user choose the next step.

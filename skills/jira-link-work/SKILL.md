---
name: jira-link-work
description: Link git commits, branches, or pull requests to Jira issues via structured comments
---

# Jira Link Work

Link development artifacts (commits, branches, PRs) to Jira issues by posting structured comments that create traceability between code and tickets.

## When to use

- User says "link my commit to PROJ-123"
- User says "post my progress on PROJ-456"
- User says "link this PR to PROJ-789"
- User says "update PROJ-123 with my branch"
- User says "log what I've done on PROJ-101"
- User says "connect my work to the ticket"
- After completing a feature, to record what was done

## How to respond

### Link current branch + recent commits

When the user says "link my work to PROJ-123":

1. Get the current branch name from git.
2. Get recent commits on this branch (compared to main/development).
3. Format and post a structured comment to the issue.

**Comment format:**

```
🔗 Development Progress

Branch: feat/PROJ-123-fix-login-css
Base: development

Commits:
• abc1234 — Fix responsive breakpoint for mobile nav
• def5678 — Add unit tests for nav component
• ghi9012 — Update snapshot tests

Files changed: 5 files (+120, -45)
```

### Link a specific commit

When the user says "link commit abc1234 to PROJ-123":

1. Get the commit details (hash, message, author, date, files).
2. Post a comment:

```
🔗 Commit Linked

abc1234 — Fix responsive breakpoint for mobile nav
Author: Shrey Purohit
Date: 2026-08-27 14:30 IST
Files: src/components/Nav.js, src/styles/nav.css
```

### Link a PR

When the user says "link PR #42 to PROJ-123" or provides a PR URL:

1. Extract the PR number/URL from the user's message.
2. Post a comment:

```
🔗 Pull Request

PR #42: Fix responsive breakpoint for mobile nav
URL: https://github.com/org/repo/pull/42
Status: Open
Branch: feat/PROJ-123-fix-login-css → development
```

### Post progress update

When the user says "post progress on PROJ-123":

1. Gather from git: current branch, commit count since base, files changed.
2. Ask the user for a brief status note (optional).
3. Post a structured comment:

```
📋 Progress Update

Branch: feat/PROJ-123-fix-login-css
Status: Implementation complete, tests passing
Commits: 5 since branch creation
Files changed: 8 files (+340, -120)

Notes: Login CSS fixed across mobile/tablet/desktop. All responsive breakpoints tested. Ready for review.
```

## Steps

### Step 1: Identify the issue key

Extract the Jira issue key from the user's request. If not explicitly mentioned, try to infer from the current branch name (e.g., `feat/PROJ-123-slug` → `PROJ-123`).

### Step 2: Gather git information

Use git commands to collect relevant data:
- `git branch --show-current` — current branch
- `git log --oneline <base>..HEAD` — commits on this branch
- `git diff --stat <base>..HEAD` — files changed summary
- `git show --stat <hash>` — specific commit details

### Step 3: Format the comment

Build the comment using the formats above. Always include:
- A clear emoji header (🔗 or 📋)
- Branch name
- Commit hashes (shortened) with messages
- File change summary

### Step 4: Confirm and post

Show the comment preview to the user and ask for confirmation before posting via `jira_add_comment`.

## Rules

- **Always confirm before posting** — show the exact comment that will be posted.
- **Never guess commit data** — only use what git returns.
- **Infer issue key from branch** when not explicitly provided (pattern: `feat/<KEY>-slug`).
- **Keep comments concise** — max 10 commits listed; if more, summarize with "... and N more commits".
- **Use the issue key** the user provides, not one from a different ticket.
- **Never include file contents or diffs** in comments — only metadata (paths, line counts).
- **Respect privacy** — don't include author emails, only names.

## Error handling

- **Not a git repository**: Report "Not in a git repository" and stop.
- **No commits on branch**: Report "No commits found on this branch relative to base."
- **Issue not found**: Report the Jira error clearly.
- **Branch doesn't match any issue**: Ask the user which issue to link to.
- **Comment post fails**: Report the error and show the comment so the user can post manually.

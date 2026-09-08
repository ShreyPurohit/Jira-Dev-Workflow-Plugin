---
name: jira-sprint
description: Show current sprint status, your assigned tickets, and blockers at a glance
---

# Jira Sprint

Show the current sprint's status — your tickets, team progress, and blockers — in one conversational request.

## When to use

- User says "what's my sprint looking like?"
- User says "show me my current sprint tickets"
- User says "show my sprint workload"
- User says "sprint status"
- User says "what am I working on in this sprint?"
- User says "what is blocking this sprint?"
- User says "show the current sprint board"
- User says "prepare me for sprint standup" or "what should I report in sprint standup?"

## Responsibility

This skill owns sprint-specific visibility and reporting: current sprint
tickets, sprint workload, progress, blockers, and standup preparation.

General Jira issue searching or listing without sprint context belongs to
`jira-read`.

## How to respond

### My tickets (default)

When the user asks about their own work in the current sprint:

1. Call `jira_search` with JQL: `assignee = currentUser() AND sprint in openSprints() ORDER BY status ASC, priority DESC`
2. Present grouped by status:

```
## Your Sprint Tickets

**In Progress (2)**
- PROJ-123 — Fix login page CSS [High]
- PROJ-456 — Add retry logic to API client [Medium]

**To Do (3)**
- PROJ-789 — Write unit tests for auth module [Medium]
- PROJ-101 — Update README with API docs [Low]
- PROJ-102 — Migrate config to env vars [Low]

**Done (1)**
- PROJ-050 — Remove deprecated endpoints ✓
```

### Sprint overview

When the user asks about the full sprint or team:

1. Call `jira_search` with JQL: `sprint in openSprints() AND project = <PROJECT> ORDER BY status ASC, assignee ASC`
2. Present a summary:

```
## Sprint: Sprint 14 (Aug 18 — Sep 1)

**Progress:** 8/20 done (40%)
**By status:** 8 Done · 5 In Progress · 7 To Do

**In Progress:**
- PROJ-123 — Fix login page CSS (Alice) [High]
- PROJ-456 — Add retry logic (Bob) [Medium]
...

**Blocked/Flagged:**
- PROJ-789 — Waiting on API team (flagged 3 days ago)
```

### Blockers

When the user asks "what is blocking this sprint?":

1. Search for flagged issues: `sprint in openSprints() AND project = <PROJECT> AND (labels = blocked OR labels = impediment OR flagged = impediment)`
2. Also check for issues with no status change in 3+ days: `sprint in openSprints() AND status = "In Progress" AND updated <= -3d`
3. Present as blockers list.

### Standup prep

When the user says "sprint standup" or "sprint standup prep":

1. Fetch the user's sprint issues updated in the last 24 hours: `assignee = currentUser() AND sprint in openSprints() AND updated >= -1d ORDER BY updated DESC`
2. Fetch the user's current in-progress sprint issues: `assignee = currentUser() AND sprint in openSprints() AND status = "In Progress"`
3. Format as standup:

```
## Standup for Aug 27

**Yesterday:**
- Completed PROJ-050 — Remove deprecated endpoints
- Progressed PROJ-123 — Fix login page CSS (moved to In Review)

**Today:**
- Continue PROJ-456 — Add retry logic to API client
- Start PROJ-789 — Write unit tests for auth module

**Blockers:**
- None currently
```

## Rules

- **Infer the project key** from context (current branch, recent queries, or ask the user once).
- **Use `currentUser()`** for personal queries — never guess the username.
- **Group by status** — always show the most actionable items first (In Progress > To Do > Done).
- **Show priority** only for High and above — don't clutter with Medium/Low markers on everything.
- **Require sprint scope for standup reporting** — use `sprint in openSprints()` when supported. If it is unsupported, use known sprint context; otherwise state that the active sprint could not be determined and ask for the sprint or project context. Never silently report unrelated personal work.
- **Never fabricate issue data** — only report what Jira returns.

## Error handling

- **No active sprint**: Report "No active sprint found for project X" and suggest checking the board.
- **No issues assigned**: Report clearly — "You have no tickets in the current sprint."
- **Project key unknown**: Ask the user which project to query.
- **Permission error**: Report the error and suggest checking project access.

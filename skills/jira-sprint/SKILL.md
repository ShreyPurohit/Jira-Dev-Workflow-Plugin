---
name: jira-sprint
description: Show current sprint status, your assigned tickets, and blockers at a glance
---

# Jira Sprint

Show the current sprint's status — your tickets, team progress, and blockers — in one conversational request.

## When to use

- User says "what's my sprint looking like?"
- User says "show me my tickets"
- User says "sprint status"
- User says "what am I working on?"
- User says "what's blocking?"
- User says "show me the board"
- User says "standup prep" or "what should I report in standup?"

## How to respond

### My tickets (default)

When the user asks about their own work:

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

When the user asks "what's blocking?":

1. Search for flagged issues: `sprint in openSprints() AND project = <PROJECT> AND (labels = blocked OR labels = impediment OR flagged = impediment)`
2. Also check for issues with no status change in 3+ days: `sprint in openSprints() AND status = "In Progress" AND updated <= -3d`
3. Present as blockers list.

### Standup prep

When the user says "standup" or "standup prep":

1. Fetch user's issues updated in last 24h: `assignee = currentUser() AND updated >= -1d ORDER BY updated DESC`
2. Fetch user's current in-progress: `assignee = currentUser() AND status = "In Progress"`
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
- **Respect JQL limitations** — if the instance doesn't support `sprint in openSprints()`, fall back to status-based queries.
- **Never fabricate issue data** — only report what Jira returns.

## Error handling

- **No active sprint**: Report "No active sprint found for project X" and suggest checking the board.
- **No issues assigned**: Report clearly — "You have no tickets in the current sprint."
- **Project key unknown**: Ask the user which project to query.
- **Permission error**: Report the error and suggest checking project access.

---
name: jira-worklog
description: Log time spent on a Jira issue via natural language (worklog entries). Use for tracking/recording time against a ticket — NOT for status changes (use jira-update-status) or comments (use jira-comment).
license: MIT
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Worklog

Log time spent against a Jira issue through natural language, without opening the
Jira UI. This skill owns worklog (time-tracking) entries.

## When to use

- User says "log 2h on PROJ-123"
- User says "add 30m worklog to PROJ-123"
- User says "I spent 1d on PROJ-123 yesterday"
- User says "track time on PROJ-123"
- User says "record 90 minutes against PROJ-123"

## When NOT to use

- **Changing the issue's status** → use `jira-update-status` (or `jira-complete`).
- **Posting a prose comment** → use `jira-comment`. A worklog comment is a short
  note attached to the time entry, not a standalone issue comment.
- **Reading how much time is already logged** → use `jira-read`.

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Create a worklog via `addOrEditJiraIssueWorklog`, reached at runtime through the v2 deferred-tool path (`discover` → `executeWrite`) like other write tools. Default to **add / create a new entry**. Never edit an existing worklog unless the user explicitly asks to edit one; if the tool schema makes add vs edit ambiguous, resolve the create path before writing.

## How to respond

### Step 1: Parse the request

Extract: issue key, time spent, optional worklog comment, optional start
date/time. Infer the issue key from the current branch (`feat/<KEY>-slug`) only
when the user omits it and a branch key is unambiguous; otherwise ask.

### Step 2: Normalise the duration

Convert to Jira's format using `w`/`d`/`h`/`m` units (`2h`, `30m`, `1d`,
`1h 30m`). Convert plain English:

- "an hour and a half" → `1h 30m`
- "90 minutes" → `1h 30m`
- "half a day" → `4h` — **assumption only** (not a universal Jira setting); echo it so the user can correct (e.g. their day may be 8h)

### Step 3: Confirm before writing

A worklog is a mutation. Show the parsed values and ask before logging:

```
Issue:    PROJ-123
Time:     1h 30m
Date:     today (2026-09-25) unless you specify otherwise
Comment:  (none)
Log this worklog?
```

### Step 4: Log it (create, don't edit)

On confirmation, call `addOrEditJiraIssueWorklog` to **create a new worklog** with
the issue key, normalised duration, and `cloudId` (plus optional comment /
`started` timestamp). Do not pass an existing worklog id or otherwise edit a prior
entry unless the user explicitly asked to change a specific worklog.

### Step 5: Verify and report

Report the entry, and the running total if the response returns it:

```
✅ Logged 1h 30m on PROJ-123 (total logged: 4h).
```

## Important rules

- **Always confirm before logging** — it is a write.
- **Create by default; never edit accidentally.** A request to "log" / "track" /
  "record" time always creates a **new** worklog. Edit an existing entry only when
  the user explicitly asks (e.g. "edit that worklog to 2h"). If add vs edit is
  ambiguous in the tool schema, resolve the create path before writing.
- **Never invent a duration.** If the user's time is ambiguous ("some time", "a
  while"), ask for a concrete value instead of guessing.
- **Default the date to today** unless the user gives one; always echo the date
  you used so a wrong assumption is visible.
- **Echo unit assumptions** (e.g. "half a day" → `4h`, or "a day" → `1d` as an
  8h working-day convention) so the user can correct them — these are assumptions,
  not fixed Jira rules.
- **Never include secrets, tokens, or sensitive paths** in a worklog comment.

## Error handling

- **Invalid duration format** → show Jira's accepted format and the value you tried.
- **Issue not found** → report the key that failed; do not guess a different key.
- **Worklog disabled on the project / permission error** → relay the Jira error,
  do not retry silently.
- **Ambiguous time** → ask for a concrete duration rather than assuming one.

## Example

User: "Log an hour and a half on PROJ-123 for the login fix"

Response:

1. Parse → key `PROJ-123`, duration `1h 30m`, comment "login fix", date today
2. Confirm the parsed values with the user (create new worklog — not an edit)
3. On confirmation, call `addOrEditJiraIssueWorklog` to create the entry with `cloudId`
4. Report: "✅ Logged 1h 30m on PROJ-123 (login fix)."

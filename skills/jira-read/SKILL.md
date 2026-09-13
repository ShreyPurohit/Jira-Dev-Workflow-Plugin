---
name: jira-read
description: Read, summarize, and search Jira issues through natural language. Use when looking up a ticket, extracting acceptance criteria, or listing issues with JQL.
license: MIT
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Read

Read, summarize, and search Jira issues through natural language.

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Call primary tools directly: `getJiraIssue`, `searchJiraIssuesUsingJql`.
4. For deferred operations, use `discover` then `executeRead` / `executeWrite` / `executeDestructive`.
5. Never embed credentials or Authorization headers in plugin files, tool arguments you invent, or Jira comments.

## When to use

- User mentions a Jira issue key (e.g., `PROJ-123`)
- User asks to "summarize", "show", "get", "read", or "look up" a Jira issue
- User asks "what are the acceptance criteria for PROJ-123?"
- User asks about ticket status, assignee, or details
- User asks to search or list issues (e.g., "show my open tickets")

## How to respond

### Single issue lookup

When the user mentions an issue key:

1. Call `getJiraIssue` with the issue key and `cloudId`.
2. Present a structured summary:
   - **Key & Summary** — the issue title
   - **Status** — current status and category
   - **Type** — Bug, Task, Story, Epic, etc.
   - **Priority** — priority level
   - **Assignee** — who it's assigned to
   - **Description** — summarize the description concisely
   - **Acceptance Criteria** — extract and list any ACs from the description
   - **Labels** — if present
   - **Links** — related issues if relevant

3. If the user specifically asks about acceptance criteria, extract them clearly as a numbered list.

### Search/list issues

When the user asks to search or list:

1. Construct appropriate JQL from their request. If the user named one or more project keys, restrict with `project = KEY` or `project in (KEY1, KEY2)`. Do not invent a project filter.
2. Call `searchJiraIssuesUsingJql` with the JQL and `cloudId`.
3. Present results as a concise table or list.

Common JQL patterns:

- "my open tickets" → `assignee = currentUser() AND status != Done`
- "open bugs in PROJ" → `project = PROJ AND issuetype = Bug AND status != Done`
- "recently updated" → `project = PROJ AND updated >= -7d ORDER BY updated DESC`

## Important rules

- **Never invent information.** If the Jira API does not return a field, say it's not available rather than guessing.
- **Issue keys are case-insensitive in user input** but always use uppercase when calling the API (e.g., user says "proj-123", you call with "PROJ-123").
- **Scope searches from the user's request.** There is no plugin environment variable for project filtering. When the user names projects, encode them in JQL; otherwise search within the authorized site.
- **Comments are read-only here.** If the user wants to add a comment, defer to the jira-comment skill.
- **Transitions are read-only here.** If the user asks to change status, defer to the jira-update-status skill.
- **Sprint reporting is out of scope here.** If the user asks about current sprint progress, workload, blockers, or standup preparation, defer to the jira-sprint skill.

## Error handling

- **Issue not found**: "I couldn't find issue PROJ-123. Please verify the issue key exists in your Jira instance."
- **Authentication failure**: "Jira authentication failed. Please reconnect or re-authorize the Atlassian Rovo MCP connection in your compatible client and verify that the authorized Jira Cloud account has the required permissions."
- **Missing or ambiguous cloudId**: "I need to know which Atlassian site to use. Please choose from the sites returned by the MCP connection."
- **Permission denied**: "You don't have permission to view PROJ-123. Check your Jira project access."

## Examples

User: "Summarize PROJ-123"

Response: Present the full issue summary with status, assignee, description, and any acceptance criteria.

User: "What are the acceptance criteria for PROJ-123?"

Response: Extract and present the ACs as a clean numbered list from the issue description.

User: "Show me open tasks in PROJ"

Response: Search with JQL `project = PROJ AND issuetype = Task AND status != Done` and present results.

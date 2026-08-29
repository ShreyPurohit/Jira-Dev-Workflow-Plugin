---
name: jira-read
description: Read, summarize, and search Jira issues through natural language
---

# Jira Read

Read, summarize, and search Jira issues through natural language.

## When to use

- User mentions a Jira issue key (e.g., `PROJ-123`)
- User asks to "summarize", "show", "get", "read", or "look up" a Jira issue
- User asks "what are the acceptance criteria for PROJ-123?"
- User asks about ticket status, assignee, or details
- User asks to search or list issues (e.g., "show my open tickets")

## How to respond

### Single issue lookup

When the user mentions an issue key:

1. Call `jira_get_issue` with the issue key.
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

1. Construct appropriate JQL from their request.
2. Call `jira_search` with the JQL.
3. Present results as a concise table or list.

Common JQL patterns:

- "my open tickets" → `assignee = currentUser() AND status != Done`
- "open bugs in PROJ" → `project = PROJ AND issuetype = Bug AND status != Done`
- "recently updated" → `project = PROJ AND updated >= -7d ORDER BY updated DESC`

## Important rules

- **Never invent information.** If the Jira API does not return a field, say it's not available rather than guessing.
- **Issue keys are case-insensitive in user input** but always use uppercase when calling the API (e.g., user says "proj-123", you call with "PROJ-123").
- **Respect project filters.** If `JIRA_PROJECTS_FILTER` is set, only search within those projects.
- **Comments are read-only here.** If the user wants to add a comment, defer to the jira-comment skill.
- **Transitions are read-only here.** If the user asks to change status, defer to the jira-update-status skill.

## Error handling

- **Issue not found**: "I couldn't find issue PROJ-123. Please verify the issue key exists in your Jira instance."
- **Authentication failure**: "Jira authentication failed. Please verify your JIRA_URL, JIRA_USERNAME, and JIRA_API_TOKEN environment variables are configured correctly."
- **Permission denied**: "You don't have permission to view PROJ-123. Check your Jira project access."

## Examples

User: "Summarize PROJ-123"

Response: Present the full issue summary with status, assignee, description, and any acceptance criteria.

User: "What are the acceptance criteria for PROJ-123?"

Response: Extract and present the ACs as a clean numbered list from the issue description.

User: "Show me open tasks in PROJ"

Response: Search with JQL `project = PROJ AND issuetype = Task AND status != Done` and present results.

# Jira Development Workflow — Agent Plugin

A task-oriented Jira development workflow plugin that sits between an AI coding agent and the Jira/Git tools it needs for day-to-day delivery work. It packages a structured workflow for reading issues, planning implementation, starting work, creating branches, linking development artifacts back to Jira, and checking sprint progress — without requiring raw API calls from the user.

Built on the open [Agent Plugins](https://agent-plugins.org/) specification — works with any compatible client.

## Compatible Clients

| Client                  | Install Method                                                                              |
| ----------------------- | ------------------------------------------------------------------------------------------- |
| **Kiro** (IDE/CLI/Crew) | Powers panel → Add Custom Power → Import from GitHub                                        |
| **VS Code**             | [Agent Plugins setup](https://code.visualstudio.com/docs/agent-customization/agent-plugins) |
| **Cursor**              | [Plugins docs](https://cursor.com/docs/plugins)                                             |
| **GitHub Copilot**      | [About plugins](https://docs.github.com/en/copilot/concepts/agents/about-plugins)           |
| **ChatGPT / Codex**     | [OpenAI plugins](https://developers.openai.com/plugins)                                     |

> **Note:** Each client discovers `plugin.json`, loads skills from `skills/`, and starts MCP servers from `mcp.json` — but the activation UX (keyword triggers, manual enable) varies per client.

## Quick Start

### 1. Install

**From GitHub (any compatible client):**

```
https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin
```

Point your client's "Add Plugin" / "Import from GitHub" flow at this URL.

**Or clone locally:**

```bash
git clone https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin.git
```

Then import the local directory in your client.

### 2. Configure Environment Variables

Set these environment variables (required by the MCP servers):

```bash
export JIRA_URL="https://yourcompany.atlassian.net"
export JIRA_USERNAME="your-email@company.com"
export JIRA_API_TOKEN="your-api-token-here"
```

#### How to get a Jira API token:

1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
2. Click "Create API token"
3. Give it a label (e.g., "AI Agent Plugin")
4. Copy the generated token

### 3. Verify

Open your AI client and try:

> "Show me the details of PROJ-123"

If you get issue details back, the plugin is working.

## What You Can Do

| Prompt                                           | What happens                                                         |
| ------------------------------------------------ | -------------------------------------------------------------------- |
| "Summarize PROJ-123"                             | Fetches and presents the issue with status, description, and ACs     |
| "What are the acceptance criteria for PROJ-123?" | Extracts and lists ACs from the description                          |
| "Plan PROJ-123"                                  | Creates a structured implementation plan from the issue requirements |
| "Start work on PROJ-123"                         | Moves the Jira issue into the appropriate In Progress status         |
| "Create a branch for PROJ-123"                   | Generates a Jira-aware Git branch name and creates the branch        |
| "Link my work to PROJ-123"                       | Posts a structured Jira comment linking branch/commit/PR context     |
| "What's my sprint looking like?"                 | Shows current sprint tickets, priorities, and blocker status         |
| "What's blocking?"                               | Surfaces blocked or stalled sprint items                             |
| "Move PROJ-123 to In Review"                     | Safely transitions the issue using dynamic discovery                 |
| "Comment on PROJ-123"                            | Adds a structured progress or completion comment                     |
| "Complete PROJ-123"                              | Verifies current state and transitions the issue through completion  |
| "Show me open tasks in PROJ"                     | Searches with appropriate JQL                                        |

## Plugin Structure

```text
jira-dev-workflow/
├── plugin.json                 # Agent Plugin manifest (identity + keywords)
├── mcp.json                    # MCP server configuration
├── README.md                   # This file
├── LICENSE                     # MIT license
├── .gitignore
├── skills/
│   ├── jira-read/SKILL.md          # Read/search/summarize issues
│   ├── jira-plan/SKILL.md          # Create implementation plans
│   ├── jira-start-work/SKILL.md    # Move a ticket into an in-progress status
│   ├── jira-update-status/SKILL.md # Transition issues safely
│   ├── jira-comment/SKILL.md       # Add structured comments
│   ├── jira-complete/SKILL.md      # Complete development workflows
│   ├── jira-branch/SKILL.md        # Create a Jira-aware Git branch
│   ├── jira-link-work/SKILL.md     # Link Git work back to Jira
│   └── jira-sprint/SKILL.md        # Show sprint status and blockers
```

### How it works

1. **`plugin.json`** declares the plugin identity, keywords, and version following the [Agent Plugins 1.0.0 spec](https://agent-plugins.org/specification).
2. **`mcp.json`** configures one stdio MCP server (`mcp-atlassian` for Jira API tools).
3. **Skills** are natural-language instructions (Agent Skills format) that teach the AI agent HOW to use the Jira MCP tools for specific development tasks.
4. When you mention a Jira issue key or ask about a Jira task, the appropriate skill activates and guides the agent through the correct workflow.

### Design principles

- **Task-oriented, not API-oriented.** Skills represent what developers ask ("start work on X") rather than raw API calls ("call jira_transition_issue").
- **Safety by default.** All write operations require user confirmation.
- **Dynamic discovery.** Transitions are discovered at runtime, not hardcoded — works with any Jira workflow.
- **Jira and Git remain separate capabilities.** Starting work and creating a branch are composable but independent operations.
- **No invented data.** The agent only reports what Jira or Git returns; it never guesses.
- **Portable.** Built on open standards (Agent Plugins + Agent Skills + MCP) — not locked to any single IDE.

## Skills

| Skill                | Purpose                                                        |
| -------------------- | -------------------------------------------------------------- |
| `jira-read`          | Read, summarize, and search Jira issues                        |
| `jira-plan`          | Analyze requirements and create implementation plans           |
| `jira-start-work`    | Move a Jira issue into the appropriate In Progress status      |
| `jira-update-status` | Safely transition Jira issues between statuses                 |
| `jira-comment`       | Add structured progress or completion comments to Jira         |
| `jira-complete`      | Complete Jira work with a completion comment and status update |
| `jira-branch`        | Create a Jira-aware Git branch from the issue summary          |
| `jira-link-work`     | Link Git work artifacts back to Jira                           |
| `jira-sprint`        | Show sprint progress, assignee workload, and blockers          |

## Jira ↔ Git Workflow

These capabilities are intentionally separate, but they can be composed when the user asks for both.

### Start work

```text
jira-start-work
      ↓
Jira status → In Progress
```

This skill reads the ticket, finds the correct transition dynamically, confirms before mutating Jira, and verifies the result. It does not create or inspect a Git branch.

### Create branch

```text
jira-branch
      ↓
Git branch → feat/PROJ-123-...
```

This skill reads the issue summary, generates a Jira-aware branch name, checks repository safety, asks for confirmation, and creates the branch. It does not transition the Jira issue to In Progress.

### Link development work

```text
jira-link-work
      ↓
Git work → Jira
```

This skill connects branch, commit, or PR context back to the Jira issue using a structured comment.

## Configuration Reference

### Required Environment Variables

| Variable         | Description                                               |
| ---------------- | --------------------------------------------------------- |
| `JIRA_URL`       | Jira instance URL (e.g., `https://company.atlassian.net`) |
| `JIRA_USERNAME`  | Your Jira login email                                     |
| `JIRA_API_TOKEN` | API token from Atlassian account settings                 |

### Optional

| Variable               | Description                                       | Default      |
| ---------------------- | ------------------------------------------------- | ------------ |
| `JIRA_PROJECTS_FILTER` | Comma-separated project keys to restrict searches | All projects |

These values are used by the Jira MCP tools and should match the environment where the plugin is running.

## MCP Servers

This plugin provides one MCP server:

| Server                      | Package                            | Tools         | Purpose              |
| --------------------------- | ---------------------------------- | ------------- | -------------------- |
| `mcp-atlassian`             | `mcp-atlassian==0.23.1`            | 63 Jira tools | Full Jira API access |

The MCP server requires [uv] installed (uvx command available).

```bash
# Install uv (provides uvx)
curl -LsSf https://astral.sh/uv/install.sh | sh
```

## Safety Conventions

| Operation                   | Safety level                              |
| --------------------------- | ----------------------------------------- |
| Read issue / search         | ✅ No confirmation needed                 |
| Get transitions (read-only) | ✅ No confirmation needed                 |
| Transition issue            | ⚠️ Confirms before executing              |
| Add comment                 | ⚠️ Shows preview, confirms before posting |
| Edit issue fields           | ⚠️ Confirms before executing              |
| Delete / close              | 🛑 Explicit confirmation required         |

### Transition safety

The most critical safety rule: **always match transitions by destination status (`to.name`), never by transition name.**

Jira transition names are arbitrary workflow labels — the agent discovers transitions dynamically and matches your intent against where the transition actually goes.

## Troubleshooting

### "Jira authentication failed"

Verify your environment variables are set:

```bash
echo $JIRA_URL        # Should be https://yourcompany.atlassian.net
echo $JIRA_USERNAME   # Should be your email
# Don't echo JIRA_API_TOKEN — just verify it's set
```

### "Issue not found"

- Check the issue key is correct (e.g., `PROJ-123`, not `proj-123`)
- Verify you have access to the project in Jira
- Check if `JIRA_PROJECTS_FILTER` is restricting your search

### "No transition available"

- The issue is in a status that doesn't allow the requested transition
- Use "Show me the transitions for PROJ-123" to see what's available
- Your Jira workflow may require intermediate steps

### MCP server not starting (`spawn uvx ENOENT`)

The MCP servers require `uvx` from [uv](https://github.com/astral-sh/uv):

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

After installation, restart your IDE/client.

> **⚠️ Do NOT use `npx -y mcp-atlassian`** — there is an unrelated broken npm package with the same name. The correct package is the Python one installed via `uvx`.

## Version Notes

The repository is currently on version 1.1.0 as defined in [plugin.json](plugin.json). The current documentation reflects the v1.1 architecture, including:

- Jira-aware branch creation
- Git-to-Jira linking work
- Sprint visibility and blocker reporting

Release notes for each version are documented in [CHANGELOG.md](./CHANGELOG.md).

## Support

For issues, questions, or feature requests, please open an issue on GitHub:

[GitHub Issues](https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin/issues)

## Privacy

This plugin is open-source and does not operate a hosted backend. For details on how it handles information, see [PRIVACY.md](./PRIVACY.md).

## Security

If you discover a security vulnerability, please do **not** disclose it publicly through GitHub Issues. Follow the responsible disclosure process described in [SECURITY.md](./SECURITY.md).

Never include Jira API tokens, credentials, or other sensitive information in public issues or reports.

## Contributing

We welcome contributions! For guidelines on development setup, testing, and submitting changes, see [CONTRIBUTING.md](./CONTRIBUTING.md).

Key points:

- Install `uv` for MCP server support
- Configure your Jira environment variables
- Test in your Agent Plugin client
- Follow the skill design principles (single responsibility, no overlap, safety first)
- Update documentation and CHANGELOG.md for release-relevant changes

## Standards

- [Agent Plugins 1.0.0](https://agent-plugins.org/specification) — plugin manifest and discovery
- [Agent Skills](https://agentskills.io/specification) — skill format (`SKILL.md`)
- [Model Context Protocol](https://modelcontextprotocol.io/specification) — MCP server configuration

## License

MIT — see [LICENSE](./LICENSE).

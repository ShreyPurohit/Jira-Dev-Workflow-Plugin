# Contributing to Jira Development Workflow Plugin

Thank you for your interest in contributing to this project! This guide explains how to develop, test, and contribute changes.

## Development Setup

### 1. Clone the repository

```bash
git clone https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin.git
cd Jira-Dev-Workflow-Plugin
```

### 2. Install dependencies

The Plugin uses the Model Context Protocol (MCP) via `uvx` for dynamic MCP server execution.

Ensure you have `uv` installed:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

This provides the `uvx` command used by `mcp.json`.

### 3. Configure your Jira environment

Set up your local Jira credentials as environment variables:

```bash
export JIRA_URL="https://yourinstance.atlassian.net"
export JIRA_USERNAME="your-email@company.com"
export JIRA_API_TOKEN="your-api-token"
```

See the README for instructions on obtaining a Jira API token.

### 4. Import the plugin into a compatible Agent Plugin client

- **Kiro:** Powers panel → Add Custom Power → Import from GitHub → paste the repository URL
- **VS Code:** Follow the [Agent Plugins setup guide](https://code.visualstudio.com/docs/agent-customization/agent-plugins)
- **Cursor or other clients:** Follow your client's plugin import flow

Point the client to this repository or your local clone.

### 5. Test manually

Try natural-language prompts in your Agent Plugin client to verify the Plugin works:

```
"Summarize PROJ-123"
"Start work on PROJ-456"
"Create a branch for PROJ-789"
```

## Repository Structure

```
├── plugin.json              # Agent Plugin manifest
├── mcp.json                 # MCP server configuration
├── README.md                # User-facing documentation
├── CHANGELOG.md             # Release notes and version history
├── LICENSE                  # MIT license
├── PRIVACY.md               # Privacy policy
├── CONTRIBUTING.md          # This file
├── .gitignore               # Git ignore rules
└── skills/
    ├── jira-read/SKILL.md           # Read and search Jira issues
    ├── jira-plan/SKILL.md           # Create implementation plans
    ├── jira-start-work/SKILL.md     # Transition issues to In Progress
    ├── jira-update-status/SKILL.md  # Update Jira issue status
    ├── jira-comment/SKILL.md        # Add comments to Jira issues
    ├── jira-complete/SKILL.md       # Complete Jira workflows
    ├── jira-branch/SKILL.md         # Create Git branches
    ├── jira-link-work/SKILL.md      # Link Git work to Jira
    └── jira-sprint/SKILL.md         # Show sprint status
```

### Key files

- **`plugin.json`:** Defines the plugin identity, version, and metadata according to the [Agent Plugins 1.0.0 specification](https://agent-plugins.org/specification).
- **`mcp.json`:** Configures MCP servers. The Plugin uses two:
  - `mcp-atlassian` for standard Jira API access
  - `mcp-atlassian-attachments` for downloading Jira attachments
- **`skills/*/SKILL.md`:** Each skill is a Markdown file with YAML frontmatter defining its name and description, followed by natural-language instructions for the AI agent.

## Adding or Modifying a Skill

### SKILL.md structure

Each skill file must follow this structure:

```markdown
---
name: skill-name
description: Brief description of what the skill does
---

# Skill Title

Clear, detailed description of the skill.

## When to use

- Trigger phrase 1
- Trigger phrase 2

## How to respond

### Step 1: ...

...

## Important rules

- Rule 1
- Rule 2

## Error handling

- **Error case 1:** Expected behavior
- **Error case 2:** Expected behavior

## Example

User: "example prompt"

Response: expected output
```

### Skill design principles

1. **Single responsibility:** Each skill should handle one distinct developer task.
2. **No overlap:** Verify that an existing skill doesn't already cover the behavior.
3. **Clear triggers:** Define specific user prompts that should activate the skill.
4. **Safety first:** All write operations should require user confirmation.
5. **Transparency:** Never invent or guess data; only report what the APIs return.
6. **Composability:** Skills should be independent. For example, "start work" and "create branch" are separate operations.

### Before adding a skill, consider:

- Does this represent a distinct developer workflow?
- Would combining it with an existing skill create harmful overlap?
- Is there a new MCP capability required?
- Does it preserve the Plugin's safety model (confirmation for writes, no invented data)?

## Testing

Before submitting changes, test the following:

### Read-only operations

- [ ] Can you fetch and summarize a Jira issue?
- [ ] Can you search for issues?
- [ ] Can you retrieve sprint information?
- [ ] Can you view transitions for an issue without executing them?

### Write operations

- [ ] Does the skill confirm before making changes?
- [ ] Does the skill verify the result after the change?
- [ ] Can the skill handle "already in target state" gracefully?

### Transition discovery

- [ ] Does the skill discover transitions dynamically from Jira?
- [ ] Does it match transitions by destination status (`to.name`), not by transition label?
- [ ] Does it handle workflows with missing expected transitions?

### Branch creation (if applicable)

- [ ] Does the skill warn about dirty working trees?
- [ ] Does it check for existing branches?
- [ ] Does it generate appropriate branch names?

### Git-to-Jira linking (if applicable)

- [ ] Can it extract issue keys from branch names?
- [ ] Can it post structured comments without exposing credentials?

### Error handling

- [ ] What happens if the Jira instance is unreachable?
- [ ] What happens if credentials are invalid?
- [ ] What happens if an issue doesn't exist?
- [ ] What happens if a workflow transition is unavailable?

### Cross-client testing

- Where possible, test in multiple Agent Plugin clients:
  - Kiro
  - VS Code with Copilot
  - Cursor
  - GitHub Copilot (if using Agent Plugins)

## Submitting a pull request

### Before you submit

1. **Describe what changed:** Explain what your contribution adds or fixes.
2. **Explain why:** Help reviewers understand the motivation.
3. **Update documentation:** If behavior changes, update the relevant `SKILL.md` or README.
4. **Update CHANGELOG:** If your change is release-relevant, add an entry to `CHANGELOG.md` under `[Unreleased]`.
5. **Keep changes focused:** Avoid mixing unrelated changes in a single PR.
6. **Never include credentials:** Do not commit API tokens, test Jira data, or private information.

### PR guidelines

- **Branches:** Use descriptive names: `feat/new-skill`, `fix/transition-discovery`, `docs/update-readme`
- **Commits:** Write clear, concise commit messages: `Add jira-sync skill for offline status updates`
- **Tests:** Describe how you tested the changes (manual testing in a client, specific scenarios verified)
- **Reviews:** Be open to feedback and iterate as needed

### Example PR template

```
## Description
Brief explanation of the change.

## Type of change
- [ ] New skill
- [ ] Bug fix
- [ ] Enhancement/improvement
- [ ] Documentation

## Testing
Describe how you tested this (e.g., "Tested in Kiro with PROJ-123 in To Do status, verified transition to In Progress succeeded and result was confirmed.")

## Checklist
- [ ] I have tested this locally
- [ ] I have updated documentation (SKILL.md, README, or CONTRIBUTING.md)
- [ ] I have updated CHANGELOG.md if release-relevant
- [ ] I have not included credentials or private data
```

## Licensing

By contributing to this repository, you agree that your contributions are made under the repository's MIT License. See [LICENSE](./LICENSE) for details.

## Questions or issues?

- **Bug reports or feature requests:** Open an issue on GitHub: https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin/issues
- **Questions about contributing:** Start a discussion or ask in an issue
- **Security concerns:** See [SECURITY.md](./SECURITY.md) for contact information

Thank you for contributing!

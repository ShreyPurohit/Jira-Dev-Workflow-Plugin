# Privacy Policy

**Effective Date:** August 29, 2026

## Overview

This privacy policy describes how the Jira Development Workflow Agent Plugin ("the Plugin") handles information.

## What This Plugin Does

The Jira Development Workflow Plugin is an open-source Agent Plugin that helps developers manage Jira issues and link development work through task-oriented, natural-language skills.

The Plugin:

- Runs locally through your compatible AI coding client
- Connects to your Jira instance through the official Atlassian Rovo MCP v2 server
- Does not operate its own backend servers or hosted services
- Does not collect, store, or transmit data outside of what you explicitly direct it to do

## Information Processed

When you use the Plugin, it may process information such as:

- Jira issue keys, summaries, and descriptions
- Acceptance criteria and issue details
- Comments and activity on Jira issues
- Issue statuses and available workflow transitions
- Sprint information and assignments
- Assignee and project information returned by your Jira instance
- Local Git branch names, commit messages, and pull request context (when Git-linking skills are invoked)

The Plugin processes this information to fulfill the specific tasks you request (e.g., "summarize this issue", "create a branch for this work").

**The Plugin itself does not store, collect, or retain this information.** All processing happens in memory during the immediate request and is subject to the data-handling policies of your AI client and Jira instance.

## Credentials and Authentication

### How authentication is provided

Authentication is handled by your compatible MCP/Agent Plugin client when it
connects to the official Atlassian Rovo MCP v2 server. The plugin itself does
not receive, store, or manage Jira credentials.

### Security practices

- **Never commit credentials to the repository.** The Plugin's `.gitignore` is configured to exclude `.env` and similar files.
- **Use Atlassian-supported authentication mechanisms.** Complete authorization through your compatible MCP client.
- **Revoke compromised credentials immediately.** Follow Atlassian's guidance if an authorization credential is exposed.
- **Use appropriately scoped access.** Grant the authorized Jira account only the permissions required for the requested workflows.

The Plugin does not maintain its own credential database, backend authentication service, or account management system.

## Data Storage

The repository provides no hosted backend, analytics service, telemetry system, or database of its own.

- **Local storage:** The Plugin may read/write to your local Git repository and your local file system (as needed for branch creation and reading).
- **Remote storage:** The only remote access is to your Jira instance and your Git repository hosting (e.g., GitHub), which are your responsibility.
- **No telemetry:** This repository does not collect usage analytics, crash reports, or telemetry data.

## Third-Party Services

Your use of this Plugin involves interaction with third-party services. You are responsible for understanding their privacy and data-handling practices:

### Atlassian / Jira

- The Plugin communicates with your Jira instance through the official Atlassian Rovo MCP v2 service at `https://mcp.atlassian.com/v2/mcp`.
- Jira processes and stores your issues, comments, and project data according to Atlassian's privacy policy.
- Review: https://www.atlassian.com/legal/privacy-policy

### Atlassian Rovo MCP v2

The Plugin uses Atlassian's hosted Rovo MCP v2 service over Streamable HTTP.

- Endpoint: `https://mcp.atlassian.com/v2/mcp`
- Transport: `streamable-http`

Authentication and authorization are handled by the compatible MCP client. Review Atlassian's policies and documentation for details about its service.

### Your AI Coding Client

- The Plugin runs through your AI client (VS Code with Copilot, Cursor, Kiro, etc.).
- Your client handles the execution environment, logging, and session management.
- Review the privacy policy of your AI client for details on what it collects and how it processes information.

### Git Repository Hosting

- If you use GitHub, GitLab, or similar services, review their privacy policies for how they handle your repositories and commits.

## What We (This Project) Do NOT Do

This open-source repository:

- Does **not** operate a backend server
- Does **not** collect usage data or analytics
- Does **not** store credentials or secrets
- Does **not** log or record your Jira data
- Does **not** share data with third parties beyond your explicit configuration
- Does **not** require account registration or sign-up
- Does **not** provide a hosted service tier

## Security Recommendations

1. **Client authentication:** Use the compatible MCP client's supported Atlassian authorization flow. Never hardcode credentials.
2. **Credential rotation:** Follow Atlassian guidance to rotate or revoke compromised authorization credentials.
3. **Permission scoping:** Grant the authorized Jira account only the permissions required for the workflows you use.
4. **Repository access:** Ensure your Git repositories are accessible only to authorized users. Do not commit Jira tokens or sensitive data.
5. **Audit:** Regularly audit who has access to your Jira instance and API tokens.

## Contact

For privacy concerns or questions about how this Plugin handles data, please open an issue on GitHub:

https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin/issues

## Changes to This Policy

This policy may be updated as the Plugin evolves. Changes will be documented in the repository's commit history and CHANGELOG.

---

**Note:** This privacy policy describes the Plugin only. It does not supersede the privacy policies of Atlassian, your AI client, or your Git hosting provider.

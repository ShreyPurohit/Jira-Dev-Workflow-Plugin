# Security Policy

## Reporting a Security Vulnerability

If you discover a security vulnerability in this repository, **please do not publicly disclose it through GitHub Issues or pull requests.** Instead, use GitHub's private security advisory mechanism.

### How to report privately

1. Navigate to the **Security** tab in the repository
2. Click **"Report a vulnerability"** or use the **"Advisories"** section
3. GitHub will guide you through the private disclosure process

This ensures the vulnerability can be addressed before it becomes public knowledge.

### What to include in a vulnerability report

A clear and detailed vulnerability report should include:

- **Description:** What is the vulnerability and how does it affect the Plugin?
- **Reproduction steps:** How can the vulnerability be reproduced or demonstrated?
- **Impact:** What is the potential impact (e.g., credential exposure, unauthorized Jira access)?
- **Affected versions:** Which version(s) of the Plugin are affected?
- **Suggested fix (optional):** If you have an idea for how to fix it, please share it

### What NOT to include

- **Never include actual Jira API tokens, usernames, or passwords** in your report
- **Never include real Jira issue data, attachments, or project information** from your instance
- Use placeholder values (e.g., `PROJ-123`, `your-company.atlassian.net`) instead

## Security Guidelines for Users

### Credential Management

- **Never commit API tokens or credentials to this repository or any Git repository**
- Authentication is handled by the compatible MCP client through Atlassian's supported authorization mechanisms
- Do not add Jira credentials, authorization headers, or tokens to plugin configuration
- Revoke compromised credentials promptly through Atlassian

### Jira Access Control

- Grant the authorized Jira account the **minimum required permissions**
- If the Plugin only needs read access, do not grant write permissions
- Review the client's Atlassian authorization scopes before enabling write-capable workflows
- Audit who has access to your Jira projects and authorization connection

### Local Security

- Protect your local development environment (laptop, CI/CD runner, etc.) just as you would protect any system with access to your Jira instance
- Keep your OS, IDE, and tools up to date with security patches
- Use appropriate file permissions to prevent unauthorized local access

### Third-Party Dependencies

This Plugin uses the official Atlassian Rovo MCP v2 integration over Streamable
HTTP at `https://mcp.atlassian.com/v2/mcp`. Do not add secrets to `mcp.json`.
Authentication is handled by the compatible MCP client. Review Atlassian's
security guidance and keep the client authorization current.

## Security Considerations

### What this Plugin does (and doesn't do)

- ✅ Communicates with your Jira instance through the official Atlassian Rovo MCP v2 service
- ✅ Executes locally on your machine or CI/CD environment
- ✅ Processes information only during active requests (no background collection)
- ❌ Does not operate a backend server or database
- ❌ Does not store your credentials or Jira data
- ❌ Does not operate a separate backend or credential store

### Known limitations

- **Atlassian Cloud only:** The official Rovo MCP v2 endpoint does not replace a self-hosted Jira API.
- **Transport and authorization:** The Plugin relies on the official Rovo MCP v2 HTTPS service and the compatible MCP client's authorization handling. Agent Plugins defines no portable OAuth fields.
- **No audit logging:** The Plugin does not maintain its own security audit log. Jira itself maintains access logs.
- **No rate limiting:** The Plugin defers to your Jira instance's rate limiting and authentication enforcement.
- **Local execution:** The Plugin runs on your machine or CI/CD environment. Its security depends on the security of that environment.

## Questions or concerns?

If you have security questions or concerns about the Plugin (that are not vulnerability reports), please open a discussion on GitHub:

https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin/issues

For vulnerability reports, please use the private security advisory process described above.

---

**Last updated:** September 13, 2026

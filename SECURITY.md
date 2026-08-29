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
- Use environment variables to inject credentials at runtime
- Configure your shell, CI/CD system, or IDE to provide credentials securely
- Regularly rotate API tokens and revoke compromised ones immediately

### Jira Access Control

- Use a Jira API token with the **minimum required permissions**
- If the Plugin only needs read access, do not grant write permissions
- Consider using project-specific or organization-scoped tokens where possible
- Audit who has access to your Jira instance and API tokens

### Local Security

- Protect your local development environment (laptop, CI/CD runner, etc.) just as you would protect any system with access to your Jira instance
- Keep your OS, IDE, and tools up to date with security patches
- Use appropriate file permissions to prevent unauthorized local access

### Third-Party Dependencies

This Plugin depends on:

- `mcp-atlassian` (version 0.23.1) — Jira MCP server
- `mcp-atlassian-attachments` (version 0.2.0) — attachment MCP server

These are third-party implementations. Review their documentation and security practices. Keep your MCP installations up to date.

## Security Considerations

### What this Plugin does (and doesn't do)

- ✅ Communicates with your Jira instance using APIs you configure
- ✅ Executes locally on your machine or CI/CD environment
- ✅ Processes information only during active requests (no background collection)
- ❌ Does not operate a backend server or database
- ❌ Does not store your credentials or Jira data
- ❌ Does not send data to third parties (beyond Jira, which you explicitly configure)

### Known limitations

- **No end-to-end encryption:** The Plugin communicates with Jira over HTTPS (as configured by your Jira instance). It does not add additional encryption layers.
- **No audit logging:** The Plugin does not maintain its own security audit log. Jira itself maintains access logs.
- **No rate limiting:** The Plugin defers to your Jira instance's rate limiting and authentication enforcement.
- **Local execution:** The Plugin runs on your machine or CI/CD environment. Its security depends on the security of that environment.

## Questions or concerns?

If you have security questions or concerns about the Plugin (that are not vulnerability reports), please open a discussion on GitHub:

https://github.com/ShreyPurohit/Jira-Dev-Workflow-Plugin/issues

For vulnerability reports, please use the private security advisory process described above.

---

**Last updated:** August 29, 2026

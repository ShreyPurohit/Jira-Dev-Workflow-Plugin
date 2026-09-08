# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

### Changed

- Migrated from the third-party `mcp-atlassian` stdio server to the official Atlassian Rovo MCP v2 server.
- Switched Jira connectivity to the remote Streamable HTTP endpoint.
- Authentication is now handled through the compatible MCP client.
- Updated documentation to reflect the new Atlassian Rovo MCP connection model.

## [1.1.1] - 2026-09-06

### Added

- Added responsible security vulnerability reporting guidance in `SECURITY.md`.
- Added privacy, contribution, and security documentation links to the README.

### Changed

- Clarified that Jira start-work transitions and Git branch creation are separate operations.
- Removed stale references to unavailable skills and corrected the README's MCP server documentation.
- Clarified responsibilities and trigger boundaries across the nine Jira skills.
- Refined start-work and completion workflows to avoid duplicating generic status-transition mechanics.
- Clarified the separation between Jira comments and Git-to-Jira development traceability.
- Restricted sprint reporting to sprint-specific requests.
- Clarified Jira planning responsibility and its boundaries with other workflow skills.

## [1.1.0] - 2026-08-29

### Added

- Jira-aware Git branch creation workflow
- Git-to-Jira work linking capability for branch, commit, and pull request context
- Sprint visibility and blocker reporting for active delivery work
- Expanded documentation for the plugin’s Jira and Git workflow model

### Changed

- Separated Jira status transitions from Git branch creation
- Refined the public-facing documentation to reflect the current nine-skill architecture
- Updated the plugin description and release-facing guidance for v1.1.0

### Fixed

- Corrected the repository’s MIT licensing text and release metadata
- Removed stale assumptions about the old six-skill architecture from public documentation

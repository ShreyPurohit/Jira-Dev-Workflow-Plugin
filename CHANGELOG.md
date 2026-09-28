# Changelog

All notable changes to this project will be documented in this file.

## [Unreleased]

## [2.1.1] - 2026-09-28

### Changed

- `jira-plan` and `jira-attachments`: attachment enumeration is now an explicit, unskippable step — both require calling `getJiraIssue` with `fields: ["attachment"]` and state that the default/evidence response does NOT reliably include the `attachment` array (absence of media there is not evidence of no attachments). This closes a real field failure where a plan was finalized after wrongly concluding a ticket had no attachments.
- `jira-plan`: added a pre-present attachment self-check (enumerated? relevant? read?) so the provisional-plan gate is confirmed explicitly rather than satisfied only in appearance.
- `jira-plan`: image attachments on a UI/UX ticket are now presumptively requirement-relevant — do not dismiss one by filename alone.
- `jira-attachments`: clarified preferring the returned `downloadCommand` (curl) in a shell-capable client over handing off the raw `downloadUrl`; sharpened the "download blocked" handling so a blocked byte-fetch is relayed as an environment/network constraint (with a path forward) rather than retried or fallen back to an unauthenticated fetch.

## [2.1.0] - 2026-09-25

### Added

- `jira-worklog` skill — log time against a Jira issue via natural language (backed by Rovo v2 `addOrEditJiraIssueWorklog`).
- `jira-review-prep` skill — generate a PR/MR description from acceptance criteria and the local git diff, then optionally transition to the review status.
- `jira-attachments` skill — download files attached to a Jira issue or comment through the authenticated MCP path (`downloadJiraIssueAttachment`), instead of an unauthenticated web fetch.
- Activation keywords: `worklog`, `time-tracking`, `review`, `pull-request`, `attachments`, `images`, `download`.

### Changed

- `jira-plan` now reads attached mockups/screenshots/spec docs (via the `jira-attachments` capability) before producing a plan, and refuses to finalize a plan while flagged attachments could not be read — producing a labeled provisional outline instead of inferred requirements.

## [2.0.0] - 2026-09-13

### Changed

- Migrated from the third-party `mcp-atlassian` stdio server to official Atlassian Rovo MCP v2 over Streamable HTTP (`https://mcp.atlassian.com/v2/mcp`), with client-managed OAuth.
- Renamed the plugin to `jira-dev-workflow-plugin` and set the package version to `2.0.0`.
- Updated skills and documentation for Rovo MCP v2 tools, `cloudId`, and Agent Plugins registry metadata.

### Breaking

- Atlassian Cloud only. Self-hosted / Data Center Jira and API tokens in plugin environment variables are no longer supported.

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

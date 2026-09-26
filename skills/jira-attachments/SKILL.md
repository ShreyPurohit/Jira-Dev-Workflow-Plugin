---
name: jira-attachments
description: Download files attached to a Jira issue or comment via the authenticated MCP server (screenshots, mockups, spec docs), instead of an unauthenticated web fetch. Use when the intent is "get/show me the files"; for planning that needs the attachments use jira-plan.
license: MIT
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Attachments

Retrieve files attached to a Jira issue or its comments through the
**authenticated** MCP path, so the agent can actually open ticket attachments
(screenshots, mockups, spec docs) instead of failing on an unauthenticated
`web_fetch` of a raw Jira URL.

> **Why this skill exists.** A reviewer left a comment with images the user
> explicitly said were needed to understand a fix. The agent read the comment
> text but could not fetch the images — Jira attachment URLs require
> authenticated access, and no skill routed the request through the MCP server,
> so it fell back to `web_fetch` and failed. The Rovo v2 server has a download
> tool; the plugin just never wired it into a skill. This closes that gap.

## When to use

- User says "show me the attachments on PROJ-123"
- User says "download the images from PROJ-123"
- User says "open the screenshot in the PROJ-123 comment"
- User says "get the mockup attached to PROJ-123"
- User says "there are images on the ticket — pull them so you can see them"

## When NOT to use

- **Reading the comment/issue text itself** → use `jira-read`. This skill is for
  the binary attachments, not the ticket body.
- **Posting a file back up to Jira** → out of scope (that is a separate upload
  write tool; add separately if ever needed).
- **Building an implementation plan that needs the attachments** → the planning
  intent belongs to `jira-plan`, which *consumes* this capability as a pre-step.
  Use `jira-attachments` when the user's intent is specifically "get / show me the
  files"; use `jira-plan` when the intent is "plan the ticket" (it pulls
  attachments itself). This keeps `jira-attachments` a distinct, self-contained
  download skill rather than a planning skill.

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Enumerate issue attachments via `getJiraIssue`. If comment attachments may not be included in that response, call `listJiraIssueComments` via `discover` → `executeRead` and scan those comments too.
4. Obtain a signed download URL via `downloadJiraIssueAttachment` (`discover` → `executeRead`). This is a read-only capability (`read_jira`), so no confirmation gate is required.

## How to respond

### Step 1: Enumerate attachments

Resolve `cloudId`, then call `getJiraIssue` for issue-level attachments. If you
need attachments from comments and they are not present in that response, call
`listJiraIssueComments` (`discover` → `executeRead`) and collect attachment ids /
filenames from the comments. Capture each attachment's id and filename.

### Step 2: Choose what to download

- **Standalone "show/download" intent:** when there is more than one attachment,
  list them (filename, type, size) so the user can pick — do not blindly pull
  everything.
- **Consumed by `jira-plan`:** read requirement-relevant attachments (mockups,
  screenshots, specs); ask the user when relevance is unclear; keep the plan
  provisional if a required file cannot be read. Planning must not stall forever
  on a picker when the files are needed for the plan.

### Step 3: Obtain a signed URL

Call `downloadJiraIssueAttachment` (discover → `executeRead`) for the chosen
attachment id. It returns a short-lived **signed** download URL plus a save
command — the authenticated path a raw `web_fetch` cannot replicate.

### Step 4: Save locally

Save the file using the returned command/URL into a scratch/working location and
report the local path.

### Step 5: Surface the content

- **Image** → hand the local path to the client so it can render/inspect it.
- **Document** → read it with normal file tools.

Report what was retrieved and where it landed.

## Important rules

- **Never claim to have seen an attachment you could not download.** If the
  download fails, say so plainly and ask the user to paste/describe the content —
  do NOT proceed to implement against inferred content the user flagged as
  necessary.
- **Signed URLs are short-lived** — fetch immediately after obtaining; if it
  expires, re-run `downloadJiraIssueAttachment` rather than reusing a stale URL.
- **Do not fall back to unauthenticated `web_fetch`** on a raw Jira attachment
  URL — it cannot authenticate and will fail; that failure is the reason this
  skill exists.
- **Network allowlist:** attachment downloads may reach `api.media.atlassian.com`
  and `*.frontend.public.atl-paas.net`. In a sandboxed/network-restricted client
  those hosts must be allowlisted (per Atlassian's Rovo MCP getting-started
  guidance); if the fetch is blocked there, relay that as the cause rather than a
  plugin fault.

## Error handling

- **Attachment not found / no attachments on the issue** → say so; do not invent one.
- **Signed URL expired** → re-request via `downloadJiraIssueAttachment`; do not
  retry the dead URL.
- **Download blocked (sandbox/allowlist)** → name the two Atlassian media hosts
  that need allowlisting and stop, rather than falling back to `web_fetch`.
- **Client cannot render the file type** → report the saved local path so the user
  can open it, instead of claiming to have viewed it.

## Example

User: "Download the mockup attached to PROJ-123 so you can see it"

Response:

1. Resolve `cloudId`, call `getJiraIssue` (and `listJiraIssueComments` if needed) → find the mockup attachment id + filename
2. Obtain a signed URL via `downloadJiraIssueAttachment` (discover → `executeRead`)
3. Save it locally and report the path
4. Hand the path to the client to render the image; if the download fails, say so and ask the user to paste it — never infer its contents

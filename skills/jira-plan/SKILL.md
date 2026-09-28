---
name: jira-plan
description: Analyze Jira issue requirements and create structured implementation plans, reading any attached mockups/screenshots/spec docs first. Use when the user asks to plan, break down, or analyze what to build for a ticket.
license: MIT
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Plan

Analyze a Jira issue's requirements and create a structured implementation plan.

## Responsibility

This skill owns Jira requirement analysis, acceptance-criteria interpretation,
implementation planning, and identifying dependencies or ambiguities.

It does not own:

- General Jira reading or searching → use `jira-read`.
- Status changes → use `jira-update-status` or `jira-start-work` for starting work.
- Comment creation → use `jira-comment`.
- Branch creation → use `jira-branch`.
- Git-to-Jira traceability → use `jira-link-work`.
- Sprint reporting → use `jira-sprint`.

It may read the Jira information needed to understand requirements, including
relevant comments when appropriate, but it does not mutate Jira or Git state.

When the issue carries image/document attachments, `jira-plan` *consumes* the
`jira-attachments` capability to read them before planning — it does not
reimplement the download mechanics (same composition pattern as `jira-complete`
over `jira-update-status`).

## When to use

- User says "create an implementation plan for PROJ-123"
- User says "analyze the requirements for PROJ-123"
- User says "plan the work for PROJ-123"
- User says "break down PROJ-123 into tasks"
- User asks to understand what needs to be built for a specific ticket

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Call `getJiraIssue` directly. If comments are needed and not included in that response, use `discover` then `executeRead` for `listJiraIssueComments`.
4. To enumerate attachments you MUST call `getJiraIssue` with `fields: ["attachment"]` — the default/evidence response does not reliably include the `attachment` array. Then read requirement-relevant ones through the `jira-attachments` capability (`downloadJiraIssueAttachment`, discovered via `discover` → `executeRead`) before finalizing the plan.
5. Never embed credentials in plugin files or Jira comments.

## How to respond

### Step 1: Read the ticket thoroughly

1. Call `getJiraIssue` with the issue key and `cloudId`. Use the fields the tool actually returns; do not assume a third-party "all fields" parameter exists.
2. Extract:
   - Summary/title
   - Full description
   - Acceptance criteria (often embedded in description)
   - Any linked issues or parent epics
   - Comments that clarify requirements
   - Priority and type

### Step 2: Read attachments (if any)

1. **You MUST enumerate attachments as a discrete tool call — call `getJiraIssue`
   with `fields: ["attachment"]` (and scan comments via `listJiraIssueComments`
   through `discover` → `executeRead`).** The default / evidence `getJiraIssue`
   view does NOT reliably return the `attachment` array, so an empty
   description-media / `comment.total: 0` response does **NOT** mean there are no
   attachments. Absence of media in the default view is not evidence of absence —
   only the explicit `fields: ["attachment"]` call is. If that call returns an
   empty attachment array on both the issue and its comments, skip to Step 3 and
   plan as normal.
2. **Select what to read for planning** — do not blindly download everything, and
   do not stop at a picker when planning needs the files:
   - Read attachments that look **relevant to requirements** (mockups, screenshots
     of the bug/UI, specs, design docs referenced by the ticket or ACs).
   - **Image attachments on a UI/UX ticket are presumptively requirement-relevant
     — do not dismiss one by its filename alone.** A file named for a different
     screen may still be the design context; when in doubt, read it or ask.
   - Skip obvious noise only when relevance is genuinely clear (e.g. an avatar, a
     logo, an unrelated export).
   - **Ask the user** when relevance or necessity is unclear, rather than guessing
     or stalling indefinitely.
3. Retrieve and read the selected files via the `jira-attachments` capability
   (`downloadJiraIssueAttachment` → signed URL → save → read). Read images for
   visual requirements and documents for written spec.
4. **Gate the plan on required attachment reads.** If a **required** attachment
   could NOT be read (download blocked, unsupported type, signed-URL failure), do
   NOT present a finalized plan. Say which attachment is unread, produce at most a
   clearly labeled *provisional* outline, and ask the user to paste/describe the
   missing content. Never fill the gap with inferred requirements.

### Step 3: Analyze requirements

From the ticket data, identify:

1. **Functional requirements** — what the feature/fix must do
2. **Acceptance criteria** — specific testable conditions for done
3. **Technical scope** — which parts of the system are affected
4. **Ambiguities** — anything unclear that needs clarification
5. **Assumptions** — reasonable assumptions you're making

### Step 4: Create the implementation plan

Structure the plan as:

```markdown
## Implementation Plan: [KEY] — [Summary]

### Requirements Summary

[Concise restatement of what needs to be done]

### Acceptance Criteria

- [ ] AC-1: [criterion]
- [ ] AC-2: [criterion]
      ...

### Test Scenarios

- TS-001: [Happy path scenario]
- TS-010: [Edge case / negative scenario]
- TS-020: [Error handling scenario]
  ...

### Technical Approach

[High-level approach to implementation]

### Subtasks

1. [First concrete implementation step]
2. [Second step]
   ...

### Ambiguities / Questions

- [Anything requiring human clarification]

### Assumptions

- [Reasonable assumptions made]
```

### Step 5: Present for review

**Before presenting, confirm this attachment self-check (state the result inline):**

- Attachments enumerated via `getJiraIssue` `fields: ["attachment"]` **and**
  comments scanned? (Y/N)
- Any attachments found that are requirement-relevant? (Y/N)
- If yes — were they all successfully downloaded and read? (Y/N)
- **If any required attachment is unread, the plan is PROVISIONAL** — label it so
  and ask the user to paste/describe the missing file. Do not present it as final.

Then present the plan to the user and ask if they'd like to:

- Approve it and proceed with development
- Request changes
- Clarify any ambiguities

## Important rules

- **Test scenarios come BEFORE development.** Define what you'll test before writing code. Use IDs like TS-001, TS-010, TS-020 for traceability.
- **Do NOT guess requirements.** If the ticket description is vague or lacks detail, flag the ambiguities. Do not fill in business logic from assumptions.
- **Never finalize a plan on unread required attachments.** Identify and read requirement-relevant attachments via `jira-attachments`; ask when relevance is unclear; if a required one cannot be read, produce a clearly labeled provisional outline and ask the user to paste/describe it — never infer the missing spec.
- **Consume the attachment capability, do not reimplement it.** The download mechanics live in `jira-attachments`; `jira-plan` only invokes them (including comment-attachment discovery when needed).
- **Acceptance criteria are the contract.** Everything in the plan must trace back to either an explicit AC or a reasonable technical necessity.
- **Plans are living documents.** If the user says the plan is wrong, update it — don't defend incorrect assumptions.
- **Do not transition the ticket** during planning. Status changes happen only when the user explicitly asks to start work.

## Error handling

- **Ticket has no description**: "PROJ-123 has no description. I can only plan from the title '[title]'. Would you like to add requirements first, or should I make assumptions and flag them?"
- **Ticket is already Done**: "PROJ-123 is already in Done status. Are you reopening this work, or did you mean a different ticket?"

## Example

User: "Create an implementation plan for PROJ-123"

Response:

1. Fetch PROJ-123 details (summary, description, ACs, comments), then enumerate attachments via `getJiraIssue` with `fields: ["attachment"]` and scan comments (use `listJiraIssueComments` when comment attachments may be missing from `getJiraIssue`) — an empty default view does NOT mean there are no attachments
2. Select requirement-relevant attachments (ask if unclear); read them via `jira-attachments`; if a required one cannot be read, stop at a labeled provisional outline and ask the user to paste/describe it
3. Extract acceptance criteria and produce a structured plan with test scenarios (TS-001 etc.), subtasks, and any ambiguities — incorporating attachment content when present
4. Present for user approval (only finalize when required attachments were readable)

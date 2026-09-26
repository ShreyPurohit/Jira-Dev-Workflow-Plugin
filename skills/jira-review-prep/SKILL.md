---
name: jira-review-prep
description: Prepare a Jira issue for review — generate a PR/MR description from acceptance criteria and the local git diff, then optionally transition to the review status. Use to get a ticket review-ready; for a bare status change use jira-update-status.
license: MIT
compatibility: Requires Atlassian Cloud Jira through Atlassian Rovo MCP v2 (https://mcp.atlassian.com/v2/mcp) with client-managed OAuth.
---

# Jira Review Prep

Turn a finished piece of work into a review-ready state: generate a PR/MR
description from the ticket's acceptance criteria plus the local git diff, and
optionally transition the ticket to its review status.

## Responsibility

This skill owns review-preparation intent: mapping a git diff back to a ticket's
acceptance criteria and producing a copy-paste PR/MR description. It coordinates
`jira-update-status` for the optional transition and can hand off to
`jira-link-work` for posting git context back to the ticket.

It does not own generic status mechanics, generic comment creation, or branch
creation, and it does not commit, push, or open PRs itself.

## When to use

- User says "prepare PROJ-123 for review"
- User says "get PROJ-123 review-ready"
- User says "write a PR description for PROJ-123"
- User says "draft the pull request for PROJ-123"

## When NOT to use

- **Just changing status to a review state, no description** → use
  `jira-update-status`.
- **Posting branch/commit/PR context back to the ticket** → use `jira-link-work`.
- **Final completion (comment + Done transition)** → use `jira-complete`.
- **Creating the branch** → use `jira-branch`.

## Atlassian MCP conventions

1. Call `getAccessibleAtlassianResources` first and obtain the `cloudId` for the user's Jira Cloud site. If more than one site is returned, ask which to use; do not guess.
2. Pass that `cloudId` on every subsequent Jira tool call.
3. Call `getJiraIssue` to read the issue.
4. For the optional review transition, follow the `jira-update-status` procedure (discover → `executeRead` for `listJiraIssueTransitions`, match by destination `to.name`, confirm, `transitionJiraIssue`, verify).

## How to respond

### Step 1: Read the ticket

Call `getJiraIssue` with the issue key and `cloudId`. Extract summary,
description, and acceptance criteria.

### Step 2: Gather local git context (read-only)

1. Determine the **base branch** from repository context (e.g. remote default /
   upstream tracking) or the user's request. If it cannot be identified
   unambiguously, **ask the user** which base to use — never assume `main` or
   `development`.
2. Collect:
   - current branch: `git branch --show-current`
   - commits since base: `git log --oneline <confirmed-base>..HEAD`
   - changed-files summary: `git diff --stat <confirmed-base>...HEAD`
   - **actual patch**: `git diff <confirmed-base>...HEAD`
3. Inspect the **patch** (and relevant files where needed) to decide AC coverage.
   Do **not** infer implementation from filenames or `--stat` line counts alone.

Infer the issue key from the branch name (`feat/<KEY>-slug`) if the user did not
supply it. This step reads git only — it never commits, pushes, or opens a PR.

### Step 3: Generate the PR/MR description

Map the **patch** back to the ticket's ACs:

```markdown
## <ISSUE-KEY>: <summary>

### What changed

- <one line per meaningful commit / file group>

### Acceptance criteria

- [x] <AC 1 — met by …>
- [x] <AC 2 — met by …>
- [ ] <AC not yet covered, if any>

### Testing

- <only claim run/passed when there is real evidence; otherwise "Not verified" / note for the developer>

Closes <ISSUE-KEY>
```

Mark an AC unchecked (and say so) when the patch does not clearly cover it. Never
claim an AC is met without evidence in the changes.

### Step 4: Present the description

Show the description for the user to copy into their PR/MR. Creating the PR
itself is out of scope (that is `gh`/`glab`, not a portable MCP tool).

### Step 5: Offer the transition

Ask whether to move the ticket to its review status. On yes, apply the transition
discovery, destination matching, confirmation, execution, and verification
procedure defined by `jira-update-status`:

- The target is the workflow's **review-status equivalent**.
- Match using the destination status field (commonly `to.name`), never the
  transition label.
- Always confirm before changing Jira state.
- Verify the resulting status afterward.

### Step 6: Offer to link the work

Suggest running `jira-link-work` to post the branch/commit context back to the
ticket.

## Important rules

- **No invented AC coverage.** Only tick an AC the **patch** actually supports;
  flag the rest explicitly. `--stat` alone is not enough evidence.
- **Identify the base branch** before gathering commits/diff. Use repository
  context or the user's request when unambiguous; ask the user when it isn't.
- **Do not invent test results.** Report tests as run and passed only when there
  is actual evidence (e.g. command output the user provided, or CI results you
  can see). A commit message or the presence of test files is **not** proof.
  Otherwise write `Not verified` or leave a testing note for the developer.
- **Read-only git.** This skill inspects git; it does not commit, push, or open PRs.
- **Confirm the transition separately** from generating the description — a user
  may want the description without moving the ticket.
- **Transition matching by `to.name`**, never by transition label.
- **Never include secrets, tokens, or sensitive paths** in the description.

## Error handling

- **Not a git repository** → generate the description from the ticket alone and
  note that git context was unavailable.
- **Ambiguous base branch** → ask which base to use before inspecting the patch.
- **No ACs in the ticket** → build the description from summary + patch and say the
  ticket had no explicit ACs to check against.
- **Issue not found / branch matches no key** → ask the user for the key.
- **No available transition to the review status** → present the description
  anyway, list the available destinations, and ask which the user wants.

## Example

User: "Prepare PROJ-123 for review"

Response:

1. Fetch PROJ-123 → summary, description, ACs
2. Confirm base branch (ask if ambiguous); read commits, `--stat`, and the full `git diff <base>...HEAD` patch
3. Generate a PR description mapping the patch to each AC (marking uncovered ACs); Testing = evidence only or "Not verified"
4. Present it to copy; then offer to transition PROJ-123 to its review status
5. On confirmation, apply the `jira-update-status` transition procedure for the review destination, verify, and offer `jira-link-work`

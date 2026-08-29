---
name: jira-plan
description: Analyze Jira issue requirements and create structured implementation plans
---

# Jira Plan

Analyze a Jira issue's requirements and create a structured implementation plan.

## When to use

- User says "create an implementation plan for PROJ-123"
- User says "analyze the requirements for PROJ-123"
- User says "plan the work for PROJ-123"
- User says "break down PROJ-123 into tasks"
- User asks to understand what needs to be built for a specific ticket

## How to respond

### Step 1: Read the ticket thoroughly

1. Call `jira_get_issue` with the issue key, requesting all fields including description and comments.
2. Extract:
   - Summary/title
   - Full description
   - Acceptance criteria (often embedded in description)
   - Any linked issues or parent epics
   - Comments that clarify requirements
   - Priority and type

### Step 2: Analyze requirements

From the ticket data, identify:

1. **Functional requirements** — what the feature/fix must do
2. **Acceptance criteria** — specific testable conditions for done
3. **Technical scope** — which parts of the system are affected
4. **Ambiguities** — anything unclear that needs clarification
5. **Assumptions** — reasonable assumptions you're making

### Step 3: Create the implementation plan

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

### Step 4: Present for review

Present the plan to the user and ask if they'd like to:

- Approve it and proceed with development
- Request changes
- Clarify any ambiguities

## Important rules

- **Test scenarios come BEFORE development.** Define what you'll test before writing code. Use IDs like TS-001, TS-010, TS-020 for traceability.
- **Do NOT guess requirements.** If the ticket description is vague or lacks detail, flag the ambiguities. Do not fill in business logic from assumptions.
- **Acceptance criteria are the contract.** Everything in the plan must trace back to either an explicit AC or a reasonable technical necessity.
- **Plans are living documents.** If the user says the plan is wrong, update it — don't defend incorrect assumptions.
- **Do not transition the ticket** during planning. Status changes happen only when the user explicitly asks to start work.

## Error handling

- **Ticket has no description**: "PROJ-123 has no description. I can only plan from the title '[title]'. Would you like to add requirements first, or should I make assumptions and flag them?"
- **Ticket is already Done**: "PROJ-123 is already in Done status. Are you reopening this work, or did you mean a different ticket?"

## Example

User: "Create an implementation plan for PROJ-123"

Response:

1. Fetch PROJ-123 details
2. Extract the acceptance criteria from the description
3. Produce a structured plan with test scenarios (TS-001 etc.), subtasks, and any ambiguities
4. Present for user approval

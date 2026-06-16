# Ticket Body Templates

## Standard Client Ticket

```markdown
**Client:** [Client name]
**App ID:** [App ID — from references/client-ids.md, if applicable]
**Campaign ID:** [campaign ID if applicable — omit if not relevant]

**Email Sender (Client):** [sender email address]
**CSM:** [CSM name / email]
**Email Thread Title:** [subject line of the original email thread]

## Issue Description
[Clear description of the problem or request]

## Steps / Evidence
[Snippet from customer email, screenshot description, or triage card context]

## Impact
[Who is affected, severity, urgency]

## Expected Behavior
[What should happen]

## Additional Context
[Doc references, workaround status, related tickets, etc.]
```

---

## Feature Request (use this template for Feature Request work type)

### Step 1 — description field (same as Standard Client Ticket)

Use the Standard Client Ticket template above for the `description` field.
Include all client IDs (Org, Project, App) in the header block.

### Step 2 — After ticket is created, ask CSM to fill in custom fields separately

> ⚙️ **SETUP:** Replace `{{YOUR_FEATURE_REQUEST_FIELD_1}}` and `{{YOUR_FEATURE_REQUEST_FIELD_2}}`
> with your Jira instance's actual custom-field IDs. Use `getJiraIssueTypeMetaWithFields` to find them.
> If your Jira doesn't have feature-request custom fields, delete Step 2 and Step 3 entirely.

After creating the ticket, always prompt the CSM with these two questions one at a time:

**Question 1 — CSM Supplemental information (`{{YOUR_FEATURE_REQUEST_FIELD_1}}`)**
```
Fill in any of the items below that you know (write "omit" if unknown or not applicable):

- Competitor provides this feature: [competitor name]
- Industry common request: [industry name]
- Other clients requesting this: [Client A], [Client B]
- Related ticket: [Jira URL]
- Renewal time / Renewal meeting: [Q4 2026] / [2026.12.31]
- Total contract size: [e.g. $100k/year]
- Expected ETA to receive PM evaluation result: [e.g. end of Nov 2026]
- Expected ETA of this feature: [e.g. end of Q4 2026]
```

**Question 2 — CSM Description (`{{YOUR_FEATURE_REQUEST_FIELD_2}}`)**
```
Fill in the items below (optional items may be skipped):

Client's Business (Optional):
- Industry:
- Platform: [web / app iOS / app Android]
- Business Goals:

Client's Account Info (Optional):
- Primary product App ID / Secondary product environment ID / other IDs:
- Campaign / Segment / Module / Report name:

Business Problem Under Current Feature (Mandatory):
[What limitation or problem exists with the current feature?]

Feature Request and Use Cases (Mandatory):
- Use Case:
- Feature Request:
```

### Step 3 — Update the ticket with CSM's answers

Once the CSM provides answers, update the ticket using `editJiraIssue`:
- CSM Supplemental information → `{{YOUR_FEATURE_REQUEST_FIELD_1}}`
- CSM Description → `{{YOUR_FEATURE_REQUEST_FIELD_2}}`

Omit fields the CSM said to skip — do not leave placeholder text.

---

## Engineering / Tech Escalation (add these fields above Issue Description)

```markdown
**Org ID:** [org]
**Project ID (primary product):** [project ID] — primary product only
**App ID (primary product):** [app ID] — primary product only
**Environment ID (secondary product):** [env ID] — secondary product only
**Bot ID (chat product):** [bot ID] — chat product only
```

> Always specify whether the environment is **PRD** or **DEV**.

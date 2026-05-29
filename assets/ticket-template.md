# Ticket Body Templates

## Standard Client Ticket

```markdown
**Client:** [Client name]
**App ID:** [appid — from references/client-ids.md, if applicable]
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

### Step 2 — After ticket is created, ask CSM to fill in two custom fields separately

After creating the ticket, always prompt the CSM with these two questions one at a time:

**Question 1 — CSM Supplemental information (`customfield_15303`)**
```
아래 항목 중 알고 계신 내용을 채워주세요 (모르거나 해당 없으면 "omit"):

- Competitor provides this feature: [경쟁사 이름]
- Industry common request: [업계 이름]
- Client common request: [고객사 A], [고객사 B] ← "Raised by" 필드에도 태그 가능
- Related ticket: [Jira URL] ← "Link issue"로도 연결 가능
- Renewal time / Renewal meeting: [Q4 2025] / [2025.12.31]
- Total contract size: [예: $100k/year]
- Expected ETA to receive PM evaluation result: [예: end of Nov 2025]
- Expected ETA of this feature: [예: end of Q4 2025]
```

**Question 2 — CSM Description (`customfield_18719`)**
```
아래 항목을 채워주세요 (Optional 항목은 생략 가능):

Client's Business (Optional):
- Industry:
- Platform: [web / app iOS / app Android]
- Business Goals:

Client's Account Info (Optional):
- AQ App ID / BB Bot ID / AX client name / AR project / PHX project:
- Campaign / Segment / Brain / Module / Report:

Business Problem Under Current Feature (Mandatory):
[현재 기능의 한계/문제점]

Feature Request and Use Cases (Mandatory):
- Use Case:
- Feature Request:
```

### Step 3 — Update the ticket with CSM's answers

Once CSM provides answers, update the ticket using `editJiraIssue`:
- CSM Supplemental information → `customfield_15303`
- CSM Description → `customfield_18719`

Omit fields the CSM said to skip — do not leave placeholder text.

---

## Engineering / Tech Escalation (add these fields above Issue Description)

```markdown
**Org ID:** [org]
**Project ID ([PRODUCT_A]):** [project] — [PRODUCT_A]-related only
**App ID ([PRODUCT_A]):** [appid] — [PRODUCT_A]-related only
**[PRODUCT_B] Environment ID:** [PLATFORM_ID] — [PRODUCT_B]-related only
**[PRODUCT_C] ID:** [PRODUCT_C_ID] — [PRODUCT_C]-related only
```

> Always specify whether the environment is **PRD** or **DEV**.

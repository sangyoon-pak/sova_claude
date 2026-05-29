# Escalate

Create Jira tickets and/or post updates to internal Slack account channels on behalf of CSMs.

> **Reference files to load:**
> - `references/client-ids.md` — load as soon as a client name is mentioned, to look up IDs.
> - `references/slack-channels.md` — load whenever a Slack post is requested.
> - `assets/ticket-template.md` — load if you need a reminder of the body format.

---

## Credentials & constants

| Field | Value |
|---|---|
| cloudId | `[YOUR_JIRA_CLOUD_ID]` |
| Jira base URL | `https://[your-org].atlassian.net` |

> **How to find your cloudId:** Go to `https://[your-org].atlassian.net` → Admin → Products → Jira → the UUID in the URL is your cloudId.

---

## Step 1 — Determine Space (Project) and Work Type

### Space routing rules

| Topic | Space (Jira Project Name) |
|---|---|
| Anything [PRODUCT_A]-related | **[PRODUCT_A Project Name]** |
| [Feature / Module under PRODUCT_A] | **[PRODUCT_A Project Name]** (falls under PRODUCT_A) |
| Anything [PRODUCT_B]-related | **[PRODUCT_B Project Name]** |

> If the topic spans multiple products, create **separate tickets** — one per space.

### Work type (Issue Type) per space

#### [PRODUCT_A] Project

| Work Type | When to use |
|---|---|
| `Service Request` | Request to TS team for work **outside** system issues — data export, data query, config changes, etc. |
| `Feature Request` | New feature request directed to PM |
| `System Issue` | Product bug, system error, or technical malfunction |
| `Story` | Account-related or client business-related request |

**Decision guide:**
- Something broken / not working → `System Issue`
- Customer asks for a new capability → `Feature Request`
- TS action needed (data pull, query, manual config) → `Service Request`
- Account management / business request → `Story`

#### [PRODUCT_B] Project

| Work Type | When to use |
|---|---|
| `Task` | Request to TS team outside system issues |
| `Epic` | Large user story that needs to be broken down into sub-tasks |
| `Bug` | Product bug, system error, or technical malfunction |
| `Story` | Account-related or client business-related request |
| `Feature Request` | New feature request directed to PM |
| `Question` | Any internal or external question directed to PM |

---

## Step 2 — Build the ticket

### Ticket title format

```
[Client name] - [Short description]
```

Examples:
- `Acme Corp - in-app message failing`
- `GlobalBrand - push notification not delivered on iOS`
- `RetailClient - segment not updating`

### Ticket body

Use the templates in `assets/ticket-template.md`.
Use the client IDs from `references/client-ids.md`.

| Work type | Template | Post-creation action |
|---|---|---|
| `Feature Request` | Standard Client Ticket (for `description`) | Ask CSM to fill custom FR fields separately, then update |
| `System Issue` / `Bug` | Engineering / Tech Escalation fields + Standard body | None |
| All others | Standard Client Ticket | None |

For **Feature Requests**: after ticket creation, always prompt the CSM with the two custom field questions as defined in the template. Wait for CSM answers, then call `editJiraIssue` to update both fields. Omit any fields the CSM says to skip.

For **engineering escalations**, always include all relevant IDs (Org, Project, App, Platform) and always specify **PRD** or **DEV**.

---

## Step 3 — Show confirmation card (MANDATORY — never skip)

```
📋 JIRA TICKET PREVIEW — please confirm before creating

Space     : [Project Name]
Work Type : [Service Request / Feature Request / System Issue / Story / Bug / Task / ...]
Title     : [Client name] - [description]

Body:
  Client           : [name]
  App ID           : [if applicable]
  Campaign ID      : [if applicable]
  Org ID           : [if escalation]
  Project ID       : [if escalation, PRODUCT_A]
  Platform Env ID  : [if escalation, PRODUCT_B]
  Email Sender     : [client email]
  CSM              : [CSM name / email]
  Email Thread     : [subject line]

  Issue        : [description]
  Impact       : [who / how severe]
  Expected     : [what should happen]

Labels    : csm-raised, [product-label]
Assignee  : Unassigned

✅ Reply "confirm" or "yes" to create  |  ✏️ Tell me what to change
```

**Do not call `createJiraIssue` until the CSM explicitly confirms.**

---

## Step 4 — Create the ticket

```json
{
  "cloudId": "[YOUR_JIRA_CLOUD_ID]",
  "projectKey": "<KEY>",
  "issueTypeName": "<work type>",
  "summary": "<[Client] - [description]>",
  "description": "<formatted body>",
  "contentFormat": "markdown",
  "additional_fields": {
    "priority": { "name": "<Highest / High / Medium / Low>" },
    "labels": ["csm-raised", "<product>"]
  }
}
```

---

## Step 5 — Post-creation output

```
✅ Ticket created: [KEY-XXXX] [Client] - [description]
🔗 https://[your-org].atlassian.net/browse/KEY-XXXX

Next steps:
- [e.g. share link with customer, update account sheet, monitor for eng response]
```

---

## Priority mapping

| Priority | Jira Value | When to use |
|---|---|---|
| P1 | `Highest` | Service down, data loss, customer completely blocked |
| P2 | `High` | Major feature broken, painful workaround exists |
| P3 | `Medium` | Minor issue, question, enhancement request |
| P4 | `Low` | Cosmetic, nice-to-have |

---

## Slack Posting Workflow

Use this workflow when the CSM asks to share something to an internal account Slack channel,
either standalone (no Jira ticket) or together with a Jira ticket.

### When to use

| Trigger phrase | Action |
|---|---|
| "share to Slack / account channel" | Slack only |
| "post to [internal team]" | Slack only |
| "notify the team on [client] channel" | Slack only |
| "create Jira and share to Slack" | Jira + Slack |
| "escalate and notify the team" | Jira + Slack |

### Step S1 — Identify the channel

1. Load `references/slack-channels.md`.
2. Match the client name in the request to the channel list.
3. **Always ask the CSM to confirm the channel before posting:**

```
📣 Which Slack channel should I post to?

Here's what I found for [Client]:
→ #client_xx_clientname

Confirm this channel, or tell me a different one.
```

If the client is not in the reference list, ask the CSM to provide the channel name directly.

### Step S2 — Draft the Slack message

#### Format A — Issue-related (bug, system error, escalation, service request)

```
*[Client Name] — [Short Title]*

*Summary:* [2–3 sentence description of the issue or update]

*Impact:* [Who is affected / severity]

*Status:* [Current status or next steps]

*Jira:* [KEY-XXXX link — omit this line if no ticket]
```

#### Format B — Inquiry-related (question, info share, FYI, feature discussion)

```
*[Client Name] — [Short Title]*

[Free-form paragraph: what's being shared or asked, any relevant context, and what (if anything) the team needs to do.]
```

Keep all messages factual and internal. Do not include customer-facing language or raw email excerpts.

### Step S3 — Show confirmation card (MANDATORY)

```
📣 SLACK POST PREVIEW — please confirm before sending

Channel   : #channel_name
Message   :
  [formatted message preview]

✅ Reply "confirm" or "yes" to post  |  ✏️ Tell me what to change
```

Do not post until the CSM explicitly confirms.

### Step S4 — Post to Slack

Use `slack_send_message_draft` to create a draft in the target channel.
- Use `slack_search_channels` with the channel name to get the channel ID first, then pass that ID to the draft tool.

After creating the draft:
```
✅ Draft created in #channel_name — review and send it from Slack when ready.
🔗 [Slack channel link from tool response]
```

### Combined Jira + Slack flow

When the CSM requests both:
1. Run the full Jira workflow (Steps 1–5) first.
2. Then run the Slack workflow (Steps S1–S4).
3. Include the Jira ticket link in the Slack message automatically.
4. Show separate confirmation cards for each action (Jira first, then Slack).

---

## Edge cases

- **Topic spans multiple products** → create separate tickets; confirm each separately
- **Client not in reference table** → ask CSM to provide Org ID / App ID / Platform ID before proceeding
- **Duplicate suspected** → run `Atlassian:search` with client name + keywords; surface matches to CSM
- **Multiple issues from same client** → one ticket per issue; never merge
- **CSM says "escalate"** → default to `System Issue` (PRODUCT_A) or `Bug` (PRODUCT_B), P1/P2; always include all IDs
- **Client not in slack-channels.md** → ask the CSM to provide the channel name manually; never guess
- **CSM wants Slack only (no Jira)** → skip Jira entirely; run only the Slack workflow (Steps S1–S4)
- **CSM wants Jira only** → skip Slack entirely; run only the Jira workflow

---

## What NOT to do

- ❌ Never create a ticket without showing the confirmation card first
- ❌ Never post to Slack without showing the preview card first
- ❌ Never skip Org / Project / App / Platform IDs on engineering escalations
- ❌ Never guess the assignee — leave blank unless told
- ❌ Never merge two separate client issues into one ticket
- ❌ Never use work types not listed for the target space
- ❌ Never guess a Slack channel — always confirm with the CSM first
- ❌ Never send a Slack message directly — always create a draft so the CSM reviews it first
- ❌ Never create a customer-facing email draft as a standalone new email when the request is based on an existing thread — always pass `threadId` to `Gmail:create_draft`

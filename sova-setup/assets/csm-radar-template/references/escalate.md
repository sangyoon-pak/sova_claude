# Escalate

Create Jira tickets and/or post updates to internal account Slack channels on behalf of your CSMs.

> **Reference files to load:**
> - `references/client-ids.md` — load as soon as a client name is mentioned, to look up IDs.
> - `references/slack-channels.md` — load whenever a Slack post is requested.
> - `assets/ticket-template.md` — load if you need a reminder of the body format.

---

## Credentials & constants

> ⚙️ **SETUP:** Filled in by the sova-setup wizard. To find manually: run
> `Atlassian:getAccessibleAtlassianResources` after connecting the Atlassian connector.

| Field | Value |
|---|---|
| cloudId | `{{YOUR_JIRA_CLOUD_ID}}` |
| Jira base URL | `https://{{YOUR_JIRA_SITE}}` |

---

## Step 1 — Determine Space (Project) and Work Type

> ⚙️ **SETUP:** Replace the project names and issue types below with the ones in your Jira
> instance. Run `getVisibleJiraProjects` to list your projects and
> `getJiraIssueTypeMetaWithFields` to list issue types per project.

### Space routing rules

| Topic | Space (Jira Project Name) |
|---|---|
| {{YOUR_PRODUCT_A}}-related issues or requests | **{{YOUR_JIRA_PROJECT_A}}** |
| {{YOUR_PRODUCT_B}}-related issues or requests | **{{YOUR_JIRA_PROJECT_B}}** |

> If the topic spans multiple products, create **separate tickets** — one per space.

### Work type (Issue Type) per space

#### {{YOUR_JIRA_PROJECT_A}}

| Work Type | When to use |
|---|---|
| `Service Request` | Request to support team for work **outside** system issues — data export, query, config changes, etc. |
| `Feature Request` | New feature request directed to PM |
| `System Issue` | Product bug, system error, or technical malfunction |
| `Story` | Account-related or client business-related request |

**Decision guide:**
- Something broken / not working → `System Issue`
- Customer asks for a new capability → `Feature Request`
- Support action needed (data pull, query, manual config) → `Service Request`
- Account management / business request → `Story`

#### {{YOUR_JIRA_PROJECT_B}}

| Work Type | When to use |
|---|---|
| `Task` | Request to support team for work outside system issues |
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
- `Acme Retail - in-app message failing`
- `Initech Corp - push notification not delivered on iOS`
- `BrandX ID - segment not updating`

### Ticket body

Use the templates in `assets/ticket-template.md`.
Use the client IDs from `references/client-ids.md`.

| Work type | Template | Post-creation action |
|---|---|---|
| `Feature Request` | Standard Client Ticket (for `description`) | Ask CSM to fill `{{YOUR_FEATURE_REQUEST_FIELD_1}}` + `{{YOUR_FEATURE_REQUEST_FIELD_2}}` separately, then update |
| `System Issue` / `Bug` | Engineering / Tech Escalation fields + Standard body | None |
| All others | Standard Client Ticket | None |

For **Feature Requests**: after ticket creation, always prompt the CSM with the two custom field
questions as defined in the template. Wait for CSM answers, then call `editJiraIssue` to update
both fields. Omit any fields the CSM says to skip.

For **engineering escalations**, always include all relevant IDs (Org, Project, App, environment)
and always specify **PRD** or **DEV**.

---

## Mentions in Jira content

Always use `contentFormat: adf` (never `markdown`) so that mentions render correctly in Jira.
Use the ADF `mention` node for any @mention of a person:

```json
{
  "type": "mention",
  "attrs": {
    "id": "{atlassian_account_id}",
    "text": "@{display name}"
  }
}
```

**Example — a paragraph with a mention:**
```json
{
  "type": "doc",
  "version": 1,
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Acme Corp confirmed their iOS app was updated. Could the support team (" },
        {
          "type": "mention",
          "attrs": {
            "id": "EXAMPLE_ACCOUNT_ID",
            "text": "@Support Team"
          }
        },
        { "type": "text", "text": ") please verify whether the issue has been resolved? Thank you!" }
      ]
    }
  ]
}
```

Never use `[~accountid:XXXX]` — that is legacy wiki markup and does not render in ADF mode.
Always split the paragraph text around the mention node.

To look up an Atlassian account ID, use `Atlassian:lookupJiraAccountId` with the person's name
or email before building the ADF body. `EXAMPLE_ACCOUNT_ID` above is illustrative only — account
IDs are never configured in advance; the skill resolves them at runtime.

---

## Step 3 — Show confirmation card (MANDATORY — never skip)

```
📋 JIRA TICKET PREVIEW — please confirm before creating

Space     : {{YOUR_JIRA_PROJECT_A}} / {{YOUR_JIRA_PROJECT_B}}
Work Type : [Service Request / Feature Request / System Issue / Story / Bug / Task / ...]
Title     : [Client name] - [description]

Body:
  Client           : [name]
  App ID           : [if applicable]
  Campaign ID      : [if applicable]
  Org ID           : [if escalation]
  Project ID       : [if escalation, primary product]
  Environment ID   : [if escalation, secondary product]
  Email Sender     : [client email]
  CSM              : [CSM name / email]
  Email Thread     : [subject line]

  Issue        : [description]
  Impact       : [who / how severe]
  Expected     : [what should happen]

Labels    : csm-raised, [product label]
Assignee  : Unassigned

✅ Reply "confirm" or "yes" to create  |  ✏️ Tell me what to change
```

**Do not call `createJiraIssue` until the CSM explicitly confirms.**

---

## Step 4 — Create the ticket

```json
{
  "cloudId": "{{YOUR_JIRA_CLOUD_ID}}",
  "projectKey": "<KEY>",
  "issueTypeName": "<work type>",
  "summary": "<[Client] - [description]>",
  "description": "<formatted body>",
  "contentFormat": "adf",
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
🔗 https://{{YOUR_JIRA_SITE}}/browse/KEY-XXXX

Next steps:
- [e.g. share link with customer, update account sheet, monitor for response]
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
| "post to the team" | Slack only |
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
→ #client_channel_name

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

[Free-form paragraph: what's being shared or asked, relevant context, and what (if anything)
the team needs to do.]
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
- Use `slack_search_channels` with the channel name to get the channel ID first, then pass
  that ID to the draft tool.

After creating the draft:
```
✅ Draft created in #channel_name — review and send it from Slack when ready.
🔗 [Slack channel link from tool response]

📌 Once you've sent the message from Slack, please paste the thread permalink here
   so I can save it to the card tracker.
   (In Slack: hover the message → ··· → Copy link)
```

Wait for the CSM to paste the thread permalink before running Step S5. If the CSM says "skip"
or doesn't provide one, proceed to Step S5 without `handoff_slack_thread`.

### Step S5 — Update card tracker (after any Jira or Slack action)

After completing any escalation action, update the card's tracker entry in Google Drive:

1. Load the latest tracker file from `CSM Radar/card-tracker.md/` (search by
   `title contains 'card-tracker-'`, pick highest by name).
2. Find the entry for the relevant `threadId` in `## Active Cards`. If not in tracker yet,
   create a new entry before updating handoff fields.
3. Update the following fields as applicable:
   - `internal_status` → `ts_investigation` · `engineering_review` · `pm_evaluation` ·
     `sales_discussion` · `csm_followup` · `legal_finance` · `pending_internal`
   - `handoff_to` → person or team name (e.g. `Support Team`, `Engineering`, `PM Team`)
   - `handoff_jira_url` → full Jira URL (e.g. `https://{{YOUR_JIRA_SITE}}/browse/PROJ-1234`)
   - `handoff_slack_channel` → channel name (e.g. `#client_channel_name`)
   - `handoff_slack_thread` → Slack thread permalink pasted by CSM (only if provided this session)
   - `handoff_at` / `last_updated` → current local datetime
4. Write the updated tracker back to Drive as a new timestamped file in
   `CSM Radar/card-tracker.md/` (parentId: `{{CARD_TRACKER_FOLDER_ID}}`).

---

### Combined Jira + Slack flow

When the CSM requests both: run Jira workflow (Steps 1–5) first, then Slack workflow (S1–S4).
Include the Jira ticket link in the Slack message automatically.
Show separate confirmation cards for each action.

---

## Edge cases

- **Topic spans multiple products** → create separate tickets; confirm each separately
- **Client not in reference table** → ask CSM to provide Org ID / App ID / environment ID
- **Duplicate suspected** → run `Atlassian:search` with client name + keywords; surface matches
- **Multiple issues from same client** → one ticket per issue; never merge
- **CSM says "escalate"** → default to `System Issue` / `Bug`, P1/P2; always include all IDs
- **Client not in slack-channels.md** → ask the CSM for the channel name; never guess
- **Slack only** → skip Jira; run Steps S1–S4 only
- **Jira only** → skip Slack; run Steps 1–5 only

---

## What NOT to do

- ❌ Never create a ticket without showing the confirmation card first
- ❌ Never post to Slack without showing the preview card first
- ❌ Never skip Org / Project / App / environment IDs on engineering escalations
- ❌ Never guess the assignee — leave blank unless told
- ❌ Never merge two separate client issues into one ticket
- ❌ Never use work types not listed for the target space
- ❌ Never guess a Slack channel — always confirm with the CSM first
- ❌ Never send a Slack message directly — always create a draft so the CSM reviews first
- ❌ Never create a standalone new email draft when the request is based on an existing thread
  — always pass `threadId` to `Gmail:create_draft`

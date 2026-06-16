# Phases 2, 3 & 4 — Field Reference

For each value: prefer fetching it for the CSM over asking. Offer the choice every time.

---

## Phase 2 — Credentials & infrastructure

### Jira cloud ID → `{{YOUR_JIRA_CLOUD_ID}}`
**Fetch it — don't ask.** Run `Atlassian:getAccessibleAtlassianResources`. It returns the site(s)
the CSM can access, each with an `id` (the cloud ID) and a `url`. If there's exactly one, use it
and confirm: "I found your Jira site — using that." If multiple, show the list and let them pick.

### Jira base URL → `{{YOUR_JIRA_SITE}}`
Comes from the same `getAccessibleAtlassianResources` response — the `url` field (e.g.
`acme.atlassian.net`). Store alongside the cloud ID.

### Card-tracker Drive folder → `{{CARD_TRACKER_FOLDER_ID}}`
**Offer to create it.** Ask: "Want me to create your memory folder in Drive, or do you already
have one?"
- *Create:* use `Google Drive:create_file` to make a folder named `card-tracker.md` under a
  `CSM Radar` parent folder. Read the returned `id` — that's the value. Confirm.
- *Existing:* ask them to paste the folder URL; the ID is the segment after `/folders/`.

### Slack workspace ID + alerts-channel ID → `{{YOUR_SLACK_WORKSPACE_ID}}`, `{{YOUR_ALERTS_CHANNEL_ID}}`
**Help find them.** Once the bot-only channel from Phase 1 exists:
- Use `Slack:slack_search_channels` with the channel name to get its channel ID (`C…`).
- The workspace ID (`T…`) comes from `app.slack.com/client/<T…>/<C…>` in the browser URL. If
  you can't derive it from tools, ask the CSM to paste the channel URL.

### Alerts channel name → `{{YOUR_ALERTS_CHANNEL_NAME}}`
The plain channel name without `#`, e.g. `csm-radar-alerts`. Capture alongside the ID.

### Timezone → `{{YOUR_TIMEZONE}}` and `{{YOUR_UTC_OFFSET}}`
Ask once. Examples: KST (UTC+9), JST (UTC+9), SGT (UTC+8), CET (UTC+1). Record both the
abbreviation and the numeric UTC offset so the derivation example in `profile.md` can be set
correctly.

---

## Phase 3 — Jira projects & work types

### Project names → `{{YOUR_JIRA_PROJECT_A}}`, `{{YOUR_JIRA_PROJECT_B}}`
**Use tools + ask.** Run `getVisibleJiraProjects` to list available projects, then ask: "Which
projects should I route tickets to?" They may have one project or several (e.g. one per product).

For each project, also collect the **issue types** they use. Run `getJiraIssueTypeMetaWithFields`
for each project to list available types (e.g. `Bug`, `Feature Request`, `Service Request`,
`Story`). Ask when to use each type. Update the routing table and decision guides in `escalate.md`
from the template accordingly.

### Feature request custom fields → `{{YOUR_FEATURE_REQUEST_FIELD_1}}`, `{{YOUR_FEATURE_REQUEST_FIELD_2}}`
Ask the CSM: "Does your Jira have custom fields for feature requests — for example, a supplemental
info field or a business context field?" Use `getJiraIssueTypeMetaWithFields` to list custom
fields. If they don't use any, remove the custom-field step from `ticket-template.md` entirely.

---

## Phase 4 — Accounts (only the CSM has these)

### Client IDs → rebuilds `references/client-ids.md`
You can't guess these. Invite them to **paste their roster or share a spreadsheet link**
("paste your accounts however you have them — a list or a spreadsheet link is fine, I'll
structure it"). For each account, collect per environment (**PRD and DEV both matter**):

- **Primary product (e.g. campaign/engagement tool):** Org ID, Project ID, App ID
- **Secondary product (e.g. CDP/data platform):** Environment ID or equivalent
- **Additional products (e.g. chat/bot tool):** Bot ID or equivalent

Not every account has every product — capture what exists. If they share a Drive spreadsheet,
read it with Google Drive tools and extract the IDs, then show them your structured version to
confirm before writing.

### Account Slack channels → rebuilds `references/slack-channels.md`
One internal channel per account. Ask them to list "account → channel name". Optionally verify
each name with `slack_search_channels` and flag any that don't resolve.

### Vendor & product names → fills `references/profile.md`
Ask: "What's your company name, and what products do you support as a CSM?" Collect:
- Vendor name (e.g. "Acme Corp")
- Product names (map to Product A, B, C in the templates)
- Official documentation base URLs per product (for web search fallback)

### Account spreadsheets (OPTIONAL) → `references/profile.md`
Only if they keep per-account tracking sheets. If they don't, tell the fill step to **delete
that section** rather than leave placeholders. Never insist on this.

---

## Atlassian account IDs — do NOT collect
csm-radar resolves @mention account IDs at runtime via `lookupJiraAccountId` from the name given
during an escalation. There is no setup value for this. Skip entirely.

# Phase 6 — Fill, Verify, Package

Do this once all values are collected and confirmed.

---

## 1. Set up a working copy

Copy the template to a writable location (never edit the asset in place), then restore its
SKILL.md (it ships as `SKILL.md.tmpl` so the package isn't read as two skills by Claude):

```bash
cp -r assets/csm-radar-template /tmp/csm-radar && cd /tmp/csm-radar
mv SKILL.md.tmpl SKILL.md
```

If the CSM uploaded their own copy (check `/mnt/user-data/uploads/`), use that path instead —
it will already have a real `SKILL.md`.

---

## 2. Simple placeholder replacements

Replace these tokens everywhere they appear across the skill files:

```bash
# Example — replace Jira cloud ID everywhere
grep -rl '{{YOUR_JIRA_CLOUD_ID}}' . | xargs sed -i "s|{{YOUR_JIRA_CLOUD_ID}}|${CLOUD_ID}|g"
# Repeat for each token below
```

| Placeholder | Value |
|---|---|
| `{{YOUR_JIRA_CLOUD_ID}}` | cloud ID from `getAccessibleAtlassianResources` |
| `{{YOUR_JIRA_SITE}}` | Jira hostname, e.g. `acme.atlassian.net` |
| `{{CARD_TRACKER_FOLDER_ID}}` | Drive folder ID |
| `{{YOUR_SLACK_WORKSPACE_ID}}` | `T…` workspace ID |
| `{{YOUR_ALERTS_CHANNEL_ID}}` | `C…` channel ID |
| `{{YOUR_ALERTS_CHANNEL_NAME}}` | channel name without `#` |
| `{{YOUR_TIMEZONE}}` | e.g. `KST` |
| `{{YOUR_UTC_OFFSET}}` | e.g. `+9` |

If the timezone isn't UTC+9, also update the derivation example in `references/profile.md`
(the `UTC + N hours` example line) to match.

---

## 3. Rebuild the Jira routing section in `escalate.md`

Replace `{{YOUR_JIRA_PROJECT_A}}` / `{{YOUR_JIRA_PROJECT_B}}` with actual project names.
Rewrite the work-type tables and decision guides to match the issue types available in the
CSM's Jira instance. If they only have one project, remove the second project's section.
Update the Step 3 confirmation card preview and Step 5 post-creation URL template to use the
real site.

---

## 4. Rebuild the account files from the roster

These aren't token substitutions — regenerate them from scratch from the confirmed roster:

**`references/client-ids.md`**
Delete ALL the `SAMPLE BLOCK` sections. For each real account, write one block:
```
### {Account Name} ({products}) · Org: `{org_id}`

| Env | Product | Name | Project / Env ID | App ID |
|---|---|---|---|---|
| PRD | {Product A} | {display name} | `{prj_id}` | `{app_id}` |
| DEV | {Product A} | {display name} DEV | `{prj_id}` | `{app_id}` |
| PRD | {Product B} | prd.{account} | `{env_id}` | — |
| DEV | {Product B} | dev.{account} | `{env_id}` | — |
```
Keep the intro note about always specifying PRD vs DEV.

**`references/slack-channels.md`**
Replace the sample rows with one row per account (`Account | channel_name | public/private`).
Keep the "Channel lookup rules" section intact.

**`references/profile.md`**
- Set vendor name, product names, and doc base URLs.
- Set timezone name and UTC offset; update the derivation example.
- If they gave spreadsheet URLs, fill the optional table. If not, delete that entire section.
- Update team rules (e.g. "Ignore quota-exceeded alerts") to match their workflow.

---

## 5. Update the ticket template if needed

If the CSM confirmed feature-request custom fields, update `assets/ticket-template.md`:
- Replace `{{YOUR_FEATURE_REQUEST_FIELD_1}}` and `{{YOUR_FEATURE_REQUEST_FIELD_2}}` with the
  actual Jira custom-field IDs.
- Translate the prompts into English (or the CSM's preferred language).
- If they have no feature-request custom fields, remove the Step 2 / Step 3 blocks from the
  Feature Request section entirely.

---

## 6. Verify — no placeholders left

```bash
grep -rn '{{' . && echo "STILL HAS PLACEHOLDERS — fix above" || echo "CLEAN — ready to package"
```

`EXAMPLE_ACCOUNT_ID` in `escalate.md` is intentional (illustrates ADF mention syntax) — leave it.
Resolve everything matching `{{…}}` before packaging.

---

## 7. Package and present

```bash
cd /tmp && zip -rq csm-radar-configured.zip csm-radar
```

Present the zip with `present_files`, then give the CSM their install instructions:

- **Web (claude.ai):** upload the `csm-radar/` folder to their Claude Project (alongside doc
  files in Project knowledge).
- **Cowork (Claude Desktop):** drop the folder into their skills directory and reload skills.

---

## 8. Hand-off summary

Close with a short recap of anything still on the CSM (e.g. "invite the bot to #alerts-channel",
"upload product doc files to both Projects", "set up the scheduled run in Cowork") and a first
test to try: **"scan my inbox."**

Remind them: if they use the Slack bot, always start a fresh conversation with `/csm-radar` so
the bot loads the skill and fetches the card tracker from Drive before responding.

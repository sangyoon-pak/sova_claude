# Profile

> ⚙️ **SETUP:** Fill in every section below. The sova-setup wizard does this for you
> interactively — run it instead of editing this file by hand.

## Vendor & products
- Vendor: {{YOUR_VENDOR}}
- Products in scope: {{YOUR_PRODUCT_A}}, {{YOUR_PRODUCT_B}}, {{YOUR_PRODUCT_C}}

## Official docs base URLs (for web search in doc lookup)

| Product | URL |
|---|---|
| {{YOUR_PRODUCT_A}} | {{YOUR_PRODUCT_A_DOCS_URL}} |
| {{YOUR_PRODUCT_B}} | {{YOUR_PRODUCT_B_DOCS_URL}} |
| Platform / Console | {{YOUR_PLATFORM_DOCS_URL}} |

## Account spreadsheets (optional)
Always ask for CSM approval before updating.

> ⚙️ **SETUP (optional):** If you keep per-account tracking sheets, list them here. If you
> don't use these, delete this entire section — it's not required.

| Account | URL |
|---|---|
| {{ACCOUNT_1_NAME}} | {{ACCOUNT_1_SHEET_URL}} |
| {{ACCOUNT_2_NAME}} | {{ACCOUNT_2_SHEET_URL}} |

## Timezone

> ⚙️ **SETUP:** Set this to your own working timezone. Update `{{YOUR_TIMEZONE}}`,
> `{{YOUR_UTC_OFFSET}}`, and the derivation example below.

**All timestamps across this skill must be in {{YOUR_TIMEZONE}} (UTC{{YOUR_UTC_OFFSET}}). No exceptions.**

This applies to:
- Card tracker filenames (`card-tracker-YYYYMMDD-HHMM.md`)
- `last_updated`, `handoff_at`, `resolved_at` fields in the tracker
- `<!-- Last updated: -->` header in tracker files
- Slack post batch run times
- Any datetime mentioned in Jira tickets or Slack messages

**How to derive local time from UTC:**
```
{{YOUR_TIMEZONE}} = UTC {{YOUR_UTC_OFFSET}} hours
Example: UTC 01:30 → {{YOUR_TIMEZONE}} 10:30
```

The assistant has no native timezone awareness. Always derive local time explicitly from UTC.
Never write a timestamp without converting first. If uncertain of the current UTC time, use
`web_search` to look it up before proceeding.

---

## Team rules

> ⚙️ **SETUP:** Edit these to match your workflow.

- Ignore quota-exceeded / automated notification emails — skip in triage, no action card needed.

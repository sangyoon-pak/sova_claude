# Client ID Reference

Use these when populating ticket bodies. Always specify PRD vs DEV.

> ⚙️ **SETUP — THIS FILE IS A TEMPLATE.**
> Replace every block below with your own accounts. One block per Org.
> - Find Org / Project / App IDs in your primary product's console.
> - Find secondary product environment IDs in that product's settings.
> - Find chat/bot IDs in your chat product's console.
> - Keep the table columns exactly as shown — the skill reads these to fill engineering escalations.
> - Delete the sample blocks once you've added your own.

---

## SAMPLE BLOCK A — single-product account

### {{CLIENT_A_NAME}} ({{YOUR_PRODUCT_A}} only) · Org: `{{CLIENT_A_ORG_ID}}`

| Env | Product | Name | Project ID | App ID |
|---|---|---|---|---|
| PRD | {{YOUR_PRODUCT_A}} | {{CLIENT_A_NAME}} | `{{CLIENT_A_PRD_PROJECT_ID}}` | `{{CLIENT_A_PRD_APP_ID}}` |
| DEV | {{YOUR_PRODUCT_A}} | {{CLIENT_A_NAME}} DEV | `{{CLIENT_A_DEV_PROJECT_ID}}` | `{{CLIENT_A_DEV_APP_ID}}` |

---

## SAMPLE BLOCK B — multi-product account (primary + secondary + chat)

### {{CLIENT_B_NAME}} ({{YOUR_PRODUCT_A}} + {{YOUR_PRODUCT_B}} + {{YOUR_PRODUCT_C}}) · Org: `{{CLIENT_B_ORG_ID}}`

| Env | Brand | Product | Name | Project / Env ID | App ID |
|---|---|---|---|---|---|
| DEV | {{BRAND}} | {{YOUR_PRODUCT_A}} | {{BRAND}} DEV | `{{B_DEV_PROJECT_ID}}` | `{{B_DEV_APP_ID}}` |
| DEV | {{BRAND}} | {{YOUR_PRODUCT_B}} | dev.{{brand}} | `{{B_DEV_ENV_ID}}` | — |
| PRD | {{BRAND}} | {{YOUR_PRODUCT_A}} | {{BRAND}} PRD | `{{B_PRD_PROJECT_ID}}` | `{{B_PRD_APP_ID}}` |
| PRD | {{BRAND}} | {{YOUR_PRODUCT_C}} | {{BRAND}} PRD | `bot-{{B_PRD_BOT_ID}}` | — |
| PRD | {{BRAND}} | {{YOUR_PRODUCT_B}} | prd.{{brand}} | `{{B_PRD_ENV_ID}}` | — |

---

## SAMPLE BLOCK C — two-product account (primary + secondary)

### {{CLIENT_C_NAME}} ({{YOUR_PRODUCT_A}} + {{YOUR_PRODUCT_B}}) · Org: `{{CLIENT_C_ORG_ID}}`

| Env | Product | Name | Project / Env ID | App ID |
|---|---|---|---|---|
| DEV | {{YOUR_PRODUCT_A}} | {{CLIENT_C_NAME}} DEV | `{{C_DEV_PROJECT_ID}}` | `{{C_DEV_APP_ID}}` |
| DEV | {{YOUR_PRODUCT_B}} | dev.{{client_c}} | `{{C_DEV_ENV_ID}}` | — |
| PRD | {{YOUR_PRODUCT_A}} | {{CLIENT_C_NAME}} PRD | `{{C_PRD_PROJECT_ID}}` | `{{C_PRD_APP_ID}}` |
| PRD | {{YOUR_PRODUCT_B}} | prd.{{client_c}} | `{{C_PRD_ENV_ID}}` | — |

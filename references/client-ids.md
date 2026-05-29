# Client ID Reference

Use these when populating ticket bodies. Always specify PRD vs DEV.

---

### [CLIENT_A] ([PRODUCT_A] only) · Org: `[ORG_ID]`

| Env | Product | Name | Project ID | App ID |
|---|---|---|---|---|
| PRD | [PRODUCT_A] | [Client A PRD name] | `[PROJECT_ID_PRD]` | `[APP_ID_PRD]` |
| STG | [PRODUCT_A] | [Client A STG name] | `[PROJECT_ID_STG]` | `[APP_ID_STG]` |

---

### [CLIENT_B] ([PRODUCT_A] + [PRODUCT_B] + [PRODUCT_C]) · Org: `[ORG_ID]`

| Env | Brand | Product | Name | Project / Platform ID | App ID |
|---|---|---|---|---|---|
| DEV | [BRAND_1] | [PRODUCT_A] | [Name DEV] | `[PROJECT_ID_DEV]` | `[APP_ID_DEV]` |
| DEV | [BRAND_1] | [PRODUCT_B] | [Platform name DEV] | `[PLATFORM_ID_DEV]` | — |
| PRD | [BRAND_1] | [PRODUCT_A] | [Name PRD] | `[PROJECT_ID_PRD]` | `[APP_ID_PRD]` |
| PRD | [BRAND_1] | [PRODUCT_C] | [Name PRD] | `[BOT_ID_PRD]` | — |
| PRD | [BRAND_1] | [PRODUCT_B] | [Platform name PRD] | `[PLATFORM_ID_PRD]` | — |

---

### [CLIENT_C] ([PRODUCT_A] + [PRODUCT_B]) · Org: `[ORG_ID]`

| Env | Brand | Product | Name | Project / Platform ID | App ID |
|---|---|---|---|---|---|
| DEV | [BRAND_1] | [PRODUCT_A] | [Name DEV] | `[PROJECT_ID_DEV]` | `[APP_ID_DEV]` |
| DEV | [BRAND_1] | [PRODUCT_B] | [Platform name DEV] | `[PLATFORM_ID_DEV]` | — |
| PRD | [BRAND_1] | [PRODUCT_A] | [Name PRD] | `[PROJECT_ID_PRD]` | `[APP_ID_PRD]` |
| PRD | [BRAND_1] | [PRODUCT_B] | [Platform name PRD] | `[PLATFORM_ID_PRD]` | — |
| DEV | [BRAND_2] | [PRODUCT_A] | [Name DEV] | `[PROJECT_ID_DEV]` | `[APP_ID_DEV]` |
| DEV | [BRAND_2] | [PRODUCT_B] | [Platform name DEV] | `[PLATFORM_ID_DEV]` | — |
| PRD | [BRAND_2] | [PRODUCT_A] | [Name PRD] | `[PROJECT_ID_PRD]` | `[APP_ID_PRD]` |
| PRD | [BRAND_2] | [PRODUCT_B] | [Platform name PRD] | `[PLATFORM_ID_PRD]` | — |

---

> **How to use this file:**
> Replace every `[PLACEHOLDER]` with your company's real values.
> Add one block per client group, with one row per brand × product × environment combination.
> Always include Org ID at the group level and specify PRD vs DEV on every row.

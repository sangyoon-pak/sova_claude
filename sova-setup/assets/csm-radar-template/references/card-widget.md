# Card Widget

Use this reference whenever you need to render action cards visually — both in **Inbox Probe** (batch of cards) and **Action Review** (single card refresh).

## When to render

- **Inbox Probe**: after collecting all card data, call `show_widget` once with all cards.
- **Action Review**: if the user says "show card", "refresh card", or asks for a structured view of the thread, render a single-card widget.

## Design principles

Match the host chat UI — clean, minimal, theme-aware. No dark backgrounds, no custom hex colors, no glow or shadow effects. Use only CSS semantic variables (`--color-background-*`, `--color-text-*`, `--color-border-*`, `--font-sans`, `--font-mono`, `--border-radius-*`).

Key patterns:
- Cards: `0.5px solid var(--color-border-tertiary)` border, `var(--border-radius-lg)` radius, default bg
- Urgency accent: `border-left: 3px solid` using danger/warning/success border vars
- Badges: small pill shapes with semantic tinted backgrounds
- Section labels: tiny uppercase monospace, muted color
- Action buttons: plain outlined, transparent bg, ↗ suffix
- Pagination: one card at a time, `#N of M` counter + ← → arrows

## Language rule

All UI text — button labels, badge text, section labels — is **always in English**, regardless of the card's email language. `sendPrompt` strings are also always in English.

## Thread status: two card types

Cards are one of two types based on `thread_status`. The type controls visual styling, badge text, section labels, and button 1. Everything else (category badge, language badge, body sections, buttons 2–4) is identical across both types.

---

### Type A — `needs_reply` (customer waiting on CSM)

**Left border accent:** urgency-driven
- `high` → `border-left: 3px solid var(--color-border-danger)`
- `medium` → `border-left: 3px solid var(--color-border-warning)`
- `low` → `border-left: 3px solid var(--color-border-success)`

**Header badges:**
- Category pill + language pill (always shown)
- Urgency badge:
  - `⏰ Reply needed today` (high)
  - `⏰ Follow up` (medium)
  - `Low priority` (low)

**Body section label:** `WHAT THEY WANT`

**Button 1:** `Draft reply ↗` → `sendPrompt('Draft reply for card #N')`

---

### Type B — `awaiting_client` (CSM replied; waiting on customer)

**Left border accent:** always muted, regardless of urgency
- `border-left: 3px solid var(--color-border-secondary)`

**Header badges:**
- Category pill + language pill (always shown)
- Status badge — replaces urgency badge entirely; always muted styling (`background: var(--color-background-secondary); color: var(--color-text-secondary)`):
  - `⏳ Awaiting client`
- Urgency badge is **not shown** for `awaiting_client` cards

**Body section label:** `WHAT WAS SENT`
- Content: 1–2 sentence summary of what the CSM sent and what response/action is expected from the customer

**Button 1:** `Follow-up draft ↗` → `sendPrompt('Draft follow-up for card #N')`

---

## What each card must show

**Header area:**
- Badge pills: category, language, status/urgency badge (per type rules above)
- Subject line (bold)
- Sender name + email, thread ID, message count, latest date — in muted small text

**Body:**
- **WHAT THEY WANT** (needs_reply) or **WHAT WAS SENT** (awaiting_client) — 1–2 sentence digest
- **CSM ANSWER** — bulleted action points
- **ESCALATE IF** + **NEXT STEPS** — side by side in tinted boxes (warning tint for escalate, neutral for next steps). Escalate box shows "N/A" if no condition.
- **SOURCES** — one row per reference, showing source type icon, title/filename, and evidence snippet:
  - Uploaded doc: `📄 filename (page N)` + snippet
  - Web: `🌐 page title` as a hyperlink + snippet

## Internal status badge

If a card has `internal_status` other than `none`, display an internal status badge in the card header alongside the urgency/status badge. Use a distinct purple tint: `background: var(--color-background-purple, #ede9fe); color: var(--color-text-purple, #6d28d9)`.

| `internal_status` value | Badge label |
|---|---|
| `ts_investigation` | 🔍 TS Investigation |
| `engineering_review` | ⚙️ Eng Review |
| `pm_evaluation` | 💡 PM Evaluation |
| `sales_discussion` | 💼 Sales Discussion |
| `csm_followup` | 📋 CSM Follow-up |
| `legal_finance` | ⚖️ Legal / Finance |
| `pending_internal` | ⏳ Pending Owner |

If any handoff fields are present, show a compact handoff info row below the digest. Render `handoff_jira_url` and `handoff_slack_thread` as clickable `<a href>` links:

```html
🔗 Handed off to [handoff_to]
   · <a href="{handoff_jira_url}" target="_blank">Jira: KEY-XXXX</a>
   · <a href="{handoff_slack_thread}" target="_blank">#handoff_slack_channel ↗</a>
```

Omit any handoff fields that are not set. If `handoff_jira_url` is present but the KEY is not separately stored, extract it from the URL (last path segment).

---

## Updated badge

If a card has `is_updated: true` (new email arrived on a previously tracked thread), display a `🔄 Updated` badge in the header next to the urgency/status badge. Use a muted blue tint: `background: var(--color-background-info); color: var(--color-text-info)`. This signals the CSM that this card reflects new activity on a known thread.

**Action buttons (always show all four):**

| # | needs_reply label | awaiting_client label | sendPrompt target |
|---|---|---|---|
| 1 | `Draft reply ↗` | `Follow-up draft ↗` | `'Draft reply for card #N'` / `'Draft follow-up for card #N'` |
| 2 | Product lookup — e.g. `Look up Product A settings ↗` | same | `'Look up [product] for card #N'` |
| 3 | `Create Jira ticket ↗` | same | `'Create a Jira ticket for card #N'` |
| 4 | `Share on Slack ↗` | same | `'Share card #N on Slack'` |
| 5 | `Mark resolved ↗` | same | `'Mark card #N as resolved'` |

Button 4 (`Share on Slack`) triggers csm-radar's Escalate mode — works both on scheduled runs and on-demand user-triggered runs. Handles channel lookup, message draft, and CSM confirmation before posting (Steps S1–S4 in `references/escalate.md`).

Button 5 (`Mark resolved`) triggers the archive flow in `references/card-tracker.md`: moves the tracker entry to Archived Cards, sets `card_status: archived`, and writes the updated tracker back to Google Drive.

# Inbox Probe

## Gmail query
`in:inbox category:primary newer_than:30d` — up to 10 threads unless user specifies otherwise.

## Step 0 — Load card tracker (blocking, before triage)

Before fetching Gmail, read `references/card-tracker.md` and follow its **Read procedure**:
1. Search Google Drive for `card-tracker.md` in the `CSM Radar/` folder.
2. Parse Active Cards into a lookup map keyed by `threadId`.
3. If file not found, start with an empty map.

This map drives the per-thread decision in Step 2 below.

## Step 1 — Fetch Gmail threads

Run the Gmail query. For each thread, collect:
- `threadId`
- `messagesTotal` (message count — available in the thread list response, no full fetch needed)
- Subject, sender, latest date

## Step 2 — Per-thread decision (no triage yet)

For each thread, consult the tracker map:

| Tracker state | Condition | Action |
|---|---|---|
| Not in tracker | threadId absent | **NEW** — proceed to triage rules below |
| In tracker, same `messagesTotal` | count matches | **UNCHANGED** — reconstruct card from tracker data. Skip triage entirely. |
| In tracker, higher `messagesTotal` | count increased | **UPDATED** — fetch only new messages. Generate delta card (see `card-tracker.md`). |

Only **NEW** and **UPDATED** threads require LLM triage. UNCHANGED threads are free.

## Triage rules (NEW threads only)

**Skip (no card):**
- Spam, receipts, marketing, automated notifications
- FYI-only mail with no follow-up needed
- Meeting-only invites with no product or account ask
- Quota-exceeded / automated system notification emails

**Create a card:**
- Customer asks for help, reports a blocker, or needs a CSM response
- Escalation signals or trackable follow-up needed

## Before rendering — doc lookup (blocking, NEW/UPDATED cards only)

For every NEW or UPDATED card that contains a technical or product claim, **read `references/doc-lookup.md` and complete its procedure before calling `show_widget`.**

Steps:
1. **Read** `references/doc-lookup.md`
2. **Local docs (Step 1):** use `project_knowledge_search` if RAG is enabled. If not, use the bash `find` + `grep` procedure described in the file.
3. **Web search (Step 2):** always runs regardless of Step 1 outcome.
4. Populate each card's `references` field with results. Use `[]` only if the card has no technical claims at all.

UNCHANGED cards skip doc lookup entirely — their references are already stored in the tracker.

Do not call `show_widget` until all NEW/UPDATED technical cards have completed this lookup.

## Card format

One Gmail thread = one card. Never merge threads. No placeholder rows for skipped items.

After processing all threads, render **all cards together as a single interactive widget** using `show_widget`. See `references/card-widget.md` for the exact HTML/JS spec.

### Thread status detection

Before assigning fields, determine `thread_status` by inspecting the **sender of the last message** in the thread:

- Last sender is Last sender is your own team's email domain (see `profile.md`) → `awaiting_client` (CSM already replied; waiting on customer)
- Last sender is any other address → `needs_reply` (customer is waiting on CSM)

### Card ordering rule

Render `needs_reply` cards first, `awaiting_client` cards last. Within each group, order by urgency descending (high → medium → low).

### Urgency cap

`awaiting_client` threads are capped at `medium` urgency maximum — never `high`. Downgrade any `high` to `medium` when `thread_status` is `awaiting_client`.

### Field reference

| Field | Value |
|---|---|
| `id` | Sequential number: 1, 2, 3… (assigned after ordering above) |
| `subject` | Email subject line |
| `sender` | Sender name / company |
| `threadId` | Gmail thread ID |
| `messagesTotal` | Message count at time of this run |
| `category` | `client_technical` · `client_non_technical` · `internal` |
| `language` | Language of the customer email |
| `thread_status` | `needs_reply` · `awaiting_client` |
| `internal_status` | `none` · `ts_investigation` · `engineering_review` · `pm_evaluation` · `sales_discussion` · `csm_followup` · `legal_finance` · `pending_internal`. Default is `none`. Set when a card is handed off internally. |
| `urgency` | `high` · `medium` · `low` (see urgency cap above) |
| `digest` | `needs_reply`: 1–2 sentence summary of what the customer wants. `awaiting_client`: 1–2 sentence summary of what the CSM sent and what response is expected. |
| `csm_bullets` | Array of grounded, actionable bullet strings |
| `escalate_if` | Condition string, or `"N/A"` |
| `next_steps` | Array of next-step strings |
| `references` | Array of reference objects — schema in `doc-lookup.md` |
| `card_language` | Language code for the card (e.g. `en` · `ko` · `ja` · `zh`) · or `internal` — drives button/badge language |
| `is_updated` | `true` if this is a delta card for a known thread; omit otherwise |

> **Button language:** all buttons, badges, and UI text are always in English. `sendPrompt` strings are always English.

> **Five buttons per card:** Draft reply / Follow-up draft · Product lookup · Jira ticket · Share on Slack · Mark resolved. Jira and Slack hand off to csm-radar Escalate mode (`references/escalate.md`). Mark resolved triggers the archive flow in `references/card-tracker.md`. See `card-widget.md` for spec.

## Step 3 — Write tracker (after show_widget)

After rendering, write the updated tracker back to Google Drive following the **Write procedure** in `references/card-tracker.md`. This includes all NEW and UPDATED entries merged with the existing active cards.

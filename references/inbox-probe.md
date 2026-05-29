# Inbox Probe

## Gmail query
`in:inbox category:primary newer_than:30d` — up to 10 threads unless user specifies otherwise.

## Triage rules

**Skip (no card):**
- Spam, receipts, marketing, automated notifications
- FYI-only mail with no follow-up needed
- Meeting-only invites with no product or account ask
- Quota-exceeded emails

**Create a card:**
- Customer asks for help, reports a blocker, or needs a CSM response
- Escalation signals or trackable follow-up needed

## Before rendering — doc lookup (blocking)

For every card that contains a technical or product claim, **read `references/doc-lookup.md` and complete its procedure before calling `show_widget`.**

Steps:
1. **Read** `references/doc-lookup.md`
2. **Local docs (Step 1):** use `project_knowledge_search` if RAG is enabled. If RAG is not enabled, fall back to the bash `find` + `grep` procedure described in the file.
3. **Web search (Step 2):** always runs regardless of Step 1 outcome.
4. Populate each card's `references` field with results. Use `[]` only if the card has no technical claims at all.

Do not call `show_widget` until all technical cards have completed this lookup.

## Card format

One Gmail thread = one card. Never merge threads. No placeholder rows for skipped items.

After triaging all threads, render **all cards together as a single interactive widget** using `show_widget`. See `references/card-widget.md` for the exact HTML/JS spec.

### Thread status detection

Before assigning fields, determine `thread_status` by inspecting the **sender of the last message** in the thread:

- Last sender is an `@[your-company].com` address → `awaiting_client` (CSM already replied; waiting on customer)
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
| `category` | `client_technical` · `client_non_technical` · `internal` |
| `language` | Language of the customer email |
| `thread_status` | `needs_reply` · `awaiting_client` |
| `urgency` | `high` · `medium` · `low` (see urgency cap above) |
| `digest` | `needs_reply`: 1–2 sentence summary of what the customer wants. `awaiting_client`: 1–2 sentence summary of what the CSM sent and what response is expected. |
| `csm_bullets` | Array of grounded, actionable bullet strings |
| `escalate_if` | Condition string, or `"N/A"` |
| `next_steps` | Array of next-step strings |
| `references` | Array of reference objects — schema in `doc-lookup.md` |
| `card_language` | `ko` · `en` · `zh` · `ja` · `internal` — drives button/badge language |

> **Button language:** all buttons, badges, and UI text are always in English. `sendPrompt` strings are always English.

> **Four buttons per card:** Draft reply / Follow-up draft · Product lookup · Jira ticket · Slack share. Jira and Slack hand off to csm-radar Escalate mode (`references/escalate.md`). See `card-widget.md` for spec.

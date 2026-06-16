# Action Review

For following up on one specific thread or action card from a previous probe.

## Scope
Stay scoped to the one thread or card the user names. Do not re-scan the inbox unless explicitly asked.

## Steps

### Step 0 — Load card tracker (blocking, before Gmail)

The card tracker is the single source of truth for all prior actions on a thread. Always load it first:

1. Search Drive for the latest `card-tracker-*.md` in folder `CSM Radar/card-tracker.md/` (parentId: `{{CARD_TRACKER_FOLDER_ID}}`). Sort by title descending, pick first.
2. Parse Active Cards into a lookup map keyed by `threadId`.
3. If the user names a client or subject (not a threadId), search Gmail for the thread first to get the `threadId`, then look it up in the tracker.
4. If found in tracker → pre-populate the card with all stored fields: `internal_status`, `handoff_to`, `handoff_jira_url`, `handoff_slack_thread`, `handoff_slack_channel`, `digest`, `csm_bullets`, `next_steps`. These are the baseline — no need to re-derive what is already known.
5. If not found in tracker → treat as a new thread; proceed with full Gmail fetch and triage.

### Step 1 — Fetch fresh Gmail context

Open the Gmail thread for any new messages since `messages_total` was last recorded in the tracker. If `messages_total` matches, the thread is unchanged — no new Gmail fetch needed unless the user explicitly asks for a refresh.

### Step 2 — Doc lookup (if technical claims)

For any technical or product claim → **read `references/doc-lookup.md`** then follow its procedure.

### Step 3 — Respond or render

Respond in prose unless the user asks for a new probe card or structured export. If rendering a card, use `references/card-widget.md` — include all five buttons and pre-fill handoff info row with tracker data.

### Step 4 — Draft a reply (if asked)

If the user asks to draft a reply, create via `Gmail:create_draft` with `threadId` so it attaches as a reply — never as a detached new email.

### Step 5 — Write tracker (if anything changed)

If `internal_status`, handoff fields, or `messages_total` changed during this review, write an updated tracker file back to Drive (new timestamped file in `CSM Radar/card-tracker.md/`).

## Language priority
1. Explicit user request
2. User's latest message language
3. Customer / snapshot language from the thread

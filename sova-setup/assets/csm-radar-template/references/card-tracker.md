# Card Tracker

Persistent memory for CSM Radar across scheduled runs. Stored in Google Drive so it survives session resets.

---

## Drive location

| Field | Value |
|---|---|
| File naming | `card-tracker-YYYYMMDD-HHMM.md` (local time) |
| Drive folder | `CSM Radar/card-tracker.md/` |
| Folder ID | `{{CARD_TRACKER_FOLDER_ID}}` |
| File search query | `title contains 'card-tracker-'` |

All per-run tracker files are saved into the `CSM Radar/card-tracker.md/` subfolder with a
local-time timestamp in the filename. This makes the latest file unambiguous by name.

---

## Read procedure (run this before Gmail triage)

1. Search Drive using `title contains 'card-tracker-'`. Filter to files with
   parentId `{{CARD_TRACKER_FOLDER_ID}}`.
2. If multiple results → sort by title descending (lexicographic sort works because format is
   `YYYYMMDD-HHMM`) and pick the first — that is the latest.
3. If found → use `Google Drive:download_file_content` with the file ID. Parse Active Cards
   into a lookup map keyed by `threadId`.
4. If not found → start with an empty map; the file will be created fresh after this run.

---

## Write procedure (run this after all cards are rendered)

1. Derive the current local timestamp in `YYYYMMDD-HHMM` format. See `references/profile.md`
   Timezone section for derivation rules.
2. **Purge stale archived entries:** drop any entry where `resolved_at` is more than 14 days
   ago. Log how many were dropped in the `Last updated` header comment.
3. Serialize the full updated tracker (active + remaining archived cards) with an updated
   `Last updated` header.
4. Create a new file titled `card-tracker-{YYYYMMDD-HHMM}.md` inside the folder using
   `Google Drive:create_file` with `parentId: {{CARD_TRACKER_FOLDER_ID}}`,
   `contentMimeType: text/plain`, and `disableConversionToGoogleType: true`.
5. Old timestamped files remain but are clearly stale by name — no cleanup needed per run.
   Periodically trash files older than 30 days manually.

> **Note:** the Google Drive MCP creates a new file on every write (no in-place update/delete).
> The timestamped-filename scheme is what keeps "latest" unambiguous.

---

## Per-thread decision logic

| Tracker state | Condition | Action |
|---|---|---|
| Not in tracker | threadId absent | **NEW** — full triage + card generation. Add to tracker. |
| In tracker, same `messagesTotal` | count matches | **UNCHANGED** — reconstruct card from tracker. No LLM call. |
| In tracker, higher `messagesTotal` | count increased | **UPDATED** — fetch only new messages. Generate delta card. Update tracker. |

---

## Tracker file format

```md
<!-- card-tracker.md — managed by CSM Radar. Do not edit manually. -->
<!-- Last updated: {ISO datetime, local time} -->

## Active Cards

### threadId:{gmail_thread_id}
- subject: {email subject}
- sender: {sender name / company}
- messages_total: {int}
- thread_status: needs_reply | awaiting_client
- internal_status: none | ts_investigation | engineering_review | pm_evaluation | sales_discussion | csm_followup | legal_finance | pending_internal
- urgency: high | medium | low
- category: client_technical | client_non_technical | internal
- language: {language code}
- digest: {1–2 sentence summary}
- csm_bullets: {bullet 1} // {bullet 2} // ...
- escalate_if: {condition or N/A}
- next_steps: {step 1} // {step 2} // ...
- handoff_to: {person or team — omit if none}
- handoff_slack_channel: {#channel-name — omit if not posted}
- handoff_slack_thread: {Slack thread permalink — omit if not posted}
- handoff_jira_url: {full Jira URL — omit if no ticket}
- handoff_at: {ISO datetime, local time — omit if none}
- last_updated: {ISO datetime, local time}
- card_status: active | archived

---

## Archived Cards
<!-- Resolved cards moved here. Delete entries older than 14 days. -->
```

Multiple active cards are separated by `---`. Every field on its own line under `### threadId:`.

---

## Delta card format

When a thread has new messages since last tracked:
- Fetch only the new messages (those beyond the previously recorded `messagesTotal`).
- Generate a card scoped to the new email(s): what changed, new ask, updated next steps.
- Visually mark with a `🔄 Updated` badge (`is_updated: true` in tracker).
- The existing card data is preserved — the delta card supplements it.
- Update `messages_total` and `last_updated` in the tracker entry.

---

## Archiving a card

**Trigger:** CSM clicks `Mark resolved ↗` (button 5).

1. Move the entry from `## Active Cards` to `## Archived Cards`.
2. Set `card_status: archived`, record `resolved_at`.
3. Run the 14-day purge — drop archived entries older than 14 days.
4. Write the updated tracker back to Drive.

---

## Slack post filter

| Condition | Post to alerts channel? |
|---|---|
| `thread_status: needs_reply` (any urgency) | ✅ Yes |
| `thread_status: awaiting_client` + `internal_status: none` | ❌ No |
| `thread_status: awaiting_client` + `internal_status` ≠ `none` | ✅ Yes — internal work in flight |

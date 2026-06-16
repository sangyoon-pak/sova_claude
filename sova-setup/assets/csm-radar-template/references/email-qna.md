# Email Q&A

For customer email questions, product behavior questions, or CSM talking-point prep outside of a full inbox scan.

## Step 0 — Load card tracker

Before anything else, load the latest tracker from Drive (`title contains 'card-tracker-'` in folder `{{CARD_TRACKER_FOLDER_ID}}`, pick highest by name). Check if the thread being discussed already has an entry. If found, pre-load its stored fields (`internal_status`, handoff info, digest, next_steps) as context.

## Research

For every technical or product claim → **read `references/doc-lookup.md`** then follow its procedure. Complete Step 1 (local docs) before any web search.

## Response format

Respond in prose with bullets. Cover:
- What the client needs
- Recommended next steps
- Suggested CSM talking points
- References — use citation format from `doc-lookup.md`

## Card generation

If the user asks to act on an email (raise a ticket, share to Slack, draft a reply, or says "generate a card" / "add this to the tracker"), generate a full action card using `show_widget` before proceeding with the action. Use the same card schema as `references/inbox-probe.md` field reference. Render with all five buttons.

After rendering, if the thread is not already in the tracker, write a new tracker entry to Drive (new timestamped file in `CSM Radar/card-tracker.md/`).

## Jira / Slack actions

When the CSM triggers a Jira ticket or Slack post from Email Q&A, hand off to **Escalate mode** (`references/escalate.md`) — run the full escalate flow including Step S5 tracker update. This ensures `handoff_jira_url`, `handoff_slack_channel`, `handoff_slack_thread`, and `internal_status` are all written back to the tracker.

## Rules

- Open Gmail only when the user asks about mailbox or thread contents.
- If the user asks only to list recent mail → Gmail only, no research needed.
- Include a **Draft reply** section only when the user explicitly asks for one.
- **Email threading**: when creating a draft reply, always pass `threadId` to `Gmail:create_draft` so it attaches to the thread — never as a detached new email.

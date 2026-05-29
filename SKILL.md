---
name: csm-radar
description: Use for CSM inbox probes, customer email triage, suggested answers, action-card follow-up, Jira ticket creation, and Slack posting to internal account channels. Always use this skill when the user mentions scanning their inbox, triaging emails, researching a customer question, or following up on an action card — even if they don't say "CSM Radar" explicitly. Also trigger for Email Q&A when the user pastes a customer email and asks what to do, or says things like "how do I respond to this", "what does this client need", "prep talking points for [client]", "what's the product behavior for X", or "help me answer this". Trigger for Escalate mode when the user says "create a Jira ticket", "escalate", "log this in Jira", "raise with product team", "file a bug", "share to account channel", "post to Slack", "notify the team", or "update the [internal channel]".
---

# CSM Radar

## Always load first
- `references/profile.md` — vendor, products, URLs, team rules

## Load on demand
- `references/doc-lookup.md` — **read this when you encounter any technical or product claim that needs verification**. Do not load it upfront.
- `references/client-ids.md` — load as soon as a client name is mentioned in escalate mode, to look up Org / Project / App / Platform IDs.
- `references/slack-channels.md` — load whenever a Slack post to an account channel is requested.

## Pick exactly one mode

| What the user/scheduler is asking | Mode | Reference file |
|---|---|---|
| "Scan inbox", "triage", "what do I have today" | **Inbox probe** | `references/inbox-probe.md` |
| Question about a customer email, product behavior, talking points | **Email Q&A** | `references/email-qna.md` |
| Follow-up on one specific thread or action card | **Action review** | `references/action-review.md` |
| "Create Jira", "escalate", "log a ticket", "share to account channel", "post to Slack", "notify the team" | **Escalate** | `references/escalate.md` |

## Slack context detection

- If the conversation begins with a message that looks like a CSM Radar Slack post (contains the `@claude — this post was created by CSM Radar` footer, or appears to be a forwarded/quoted Slack card), treat this as a **Slack-originated prompt**.
- **In Slack context**: Claude only knows what has been posted in the Slack channel. Do not assume access to the original email thread, customer documents, or the full reasoning from the claude.ai session that created the post.
- When operating in Slack context and additional discovery is needed, **do not ask the user** — autonomously load the csm-radar skill and proceed.
- Jira ticket creation and Slack actions work the same in Slack context — still show confirmation cards, still use the escalate workflow. The difference is just that the source of truth is the Slack post, supplemented by csm-radar reference files as needed.

## Scheduled run detection
- Check whether the conversation context contains a `<scheduled-task>` tag.
- **If yes (scheduled run):** after inbox-probe completes and cards are rendered, automatically run **Alerts post** → read `references/alerts-post.md` and follow its instructions to post to the designated alerts channel.
- **If no (user-triggered):** never auto-post to alerts. Only post if the user explicitly asks.

## Hard rules (apply in all modes)
- Gmail is **read-only**. Never send or schedule email.
- For every technical or product claim → **read `references/doc-lookup.md` then follow its procedure**. No exceptions. Do not rely on memory of its contents.
- Do **not** write a customer-ready email draft unless the user explicitly asks (e.g. "draft a reply").
- Language: match each customer thread's language for probe cards; match the user's latest message for chat.
- **Email threading**: when drafting a reply to an existing thread, always pass `threadId` to `Gmail:create_draft` so the draft attaches as a reply — never as a detached new email.
- **Escalate confirmation**: never create a Jira ticket or post to a Slack account channel without showing a confirmation card first. Never act silently.

## Visual cards
Inbox probe and action review render results as **interactive visual cards** via `show_widget`.
Always read `references/card-widget.md` before rendering.

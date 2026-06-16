# Phase 1 & 5 — Prerequisites

## Connectors (Phase 1)

CSM Radar needs four connectors. Ask the CSM to confirm each is connected in Claude
(Settings → Connectors). If one is missing, point them there before continuing — you can't
fetch IDs for a connector that isn't linked yet.

| Connector | CSM Radar uses it to… | Permissions to enable | Must NOT need |
|---|---|---|---|
| Gmail | read inbox threads, create draft replies | read messages/threads · create drafts | send · delete |
| Google Drive | store & read its memory (card tracker) | read files · create files | delete · org-wide |
| Slack | post the alerts summary · draft account-channel messages | read channels · post/draft · look up users & channels | admin · delete |
| Jira (Atlassian) | create & update escalation tickets | read · create & edit issues · look up accounts | project admin · delete |

Quick verification: for Atlassian, you'll run `getAccessibleAtlassianResources` in Phase 2 anyway —
if it returns a site, Jira is good to go.

## Slack Claude app + bot-only channel (Phase 1)

Two one-time Slack steps, both required for scheduled runs and for driving CSM Radar from Slack:

1. **Enable the Claude app (bot) in the workspace.** Their Slack admin may need to approve it.
   Once enabled, they can `@mention` the Claude app in any channel it's invited to.
2. **Create a private alerts channel and invite ONLY the Claude app bot** (e.g. `#csm-radar-alerts`,
   no human members). Scheduled summaries post here, and it doubles as a clean running log the bot
   can read back. In Phase 2 you'll capture this channel's ID.

If the CSM hasn't made the channel yet, walk them through it, then continue — you'll read its IDs
from the channel URL or via `slack_search_channels` once it exists.

## Product docs (Phase 5) — a pointer, not a fill-in value

CSM Radar checks product behaviour against the CSM's own documentation files.

**First, ask the doc question:**
> "Do you have your product documentation in Markdown format yet?
> If not, I can walk you through how to create it."

The most reliable way to get docs into Markdown is to scrape your official documentation website.
Use Claude Code or another AI agent tool to crawl your docs site and output `.md` files — one file
per major section. Re-run when docs change.

**Then explain how to load them for their surface:**

- **Web (claude.ai):** create a **Project**, add the Markdown doc files to the Project
  **knowledge**, and keep the configured `csm-radar` skill in that Project. CSM Radar finds the
  docs via knowledge search (`project_knowledge_search`).
- **Cowork (Claude Desktop):** place the doc files in a path-accessible location. Keep filenames
  consistent with what's listed in `references/doc-lookup.md` inside the configured skill; update
  that file if the CSM uses different names.

If they skip this, CSM Radar still works — it falls back to a web search for every product claim,
which is slower and less precise. Recommend loading the docs, but don't block setup on it.

**Where docs come from:** the Markdown files are a snapshot of the documentation at scraping time.
When the official docs change, re-scrape and replace the files. Mention this so CSMs know the
docs can drift and how to refresh them.

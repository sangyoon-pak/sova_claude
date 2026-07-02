# CSM Radar — Setup Guide

*A Claude-powered inbox copilot for enterprise CSMs.*

---

## What it does

Enterprise CSM work runs on two repetitive loops:

1. **Technical / issue email** → triage → support handoff + Jira ticket + customer reply
2. **Request email** → requirement analysis → Slack share + stakeholder tag + Jira ticket

CSM Radar automates the mechanics of both. It runs on a schedule (e.g. every 4 hours, business
hours), builds interactive action cards in Claude, and pushes Slack alerts. You confirm before
anything gets filed or posted.

| Mode | Trigger | What happens |
|---|---|---|
| **Inbox Probe** | "Scan my inbox" / scheduled | Reads Gmail, triages new/changed threads, returns action cards |
| **Email Q&A** | Paste email + "how do I answer this?" | Researches product docs + web, gives grounded answer + optional draft |
| **Action Review** | "Follow up on the Acme thread" | Restores full thread history from memory, refreshes the card |
| **Escalate** | Card button or direct ask | Builds Jira ticket or Slack post → preview → confirm → done |

Escalate runs *after* the other three — card buttons hand off to it. It always previews before
filing.

---

## The research backbone — CSM Radar's "brain"

CSM Radar grounds every product or technical claim in your own documentation before acting. Think
of it as giving the agent a NotebookLM-style knowledge base plus the tools to draft replies,
raise Jira tickets, and post to account Slack channels.

The research backbone runs on **claude.ai** and **Cowork** during inbox triage and Q&A modes.
The **Slack bot** does not triage or search docs — it looks up cards from the tracker and uses
**web search** when product lookup is needed.

| Surface | How docs are searched | Notes |
|---|---|---|
| **Claude Cowork** (Claude Desktop) | Keyword search (`grep`) over path-loaded files | Scheduled Inbox Probe; less efficient than RAG |
| **Claude Chat** (claude.ai web) | Embedding-based RAG over Project knowledge | Inbox Probe · Email Q&A · Action Review |
| **Slack bot** | No doc access | Reads the Slack card post + tracker; web search only |

On claude.ai and Cowork, CSM Radar also runs a targeted **web search** after doc lookup to confirm currency.
The Slack bot skips doc lookup and relies on web search alone — slower and less precise for product claims.

> **Important:** Cowork and claude.ai web do **not** share Project knowledge or memory. Upload
> your product docs to **both** Projects separately, and update each when docs change.

---

## The card tracker — Sova's memory

Every session (claude.ai, Cowork, Slack bot) starts with zero memory. What makes Sova feel
continuous is one file: the **card tracker** (`card-tracker-*.md`) stored in your Google Drive.
Sova reads the latest one at the start of every run and writes a fresh timestamped copy at the
end.

Each card holds what Sova worked out about a thread: the triage verdict (`digest`, `urgency`,
`next_steps`), its current state (`needs_reply` / `awaiting_client`, plus an `internal_status`
like `ts_investigation`, `engineering_review`, or `pm_evaluation`), and the message count at last
check.

Once you escalate, the card also records the full handoff trail — who it went to (`handoff_to`,
e.g. Daniel Jang (TS), Engineering, PM Team), the Jira ticket it became (`handoff_jira_url`), the
account Slack channel it was posted to (`handoff_slack_channel`), the Slack thread permalink for
that post (`handoff_slack_thread`), and when the handoff happened (`handoff_at`). So a card never
forgets where it was sent or what it turned into.

On the next run Sova diffs each thread's live message count against the stored one: unchanged
threads are rebuilt from the tracker with no re-triage, changed ones get a delta card, and
resolved cards are archived. The handoff trail keeps the loop coherent — an already-escalated
thread shows its existing Jira and Slack links instead of being escalated again, and any
`awaiting_client` card with live internal work still surfaces in `#claude-alerts` (via
`internal_status`) so the team stays informed while the reply is on hold.

**Don't hand-edit it. Don't delete the folder. It's the agent's brain.**



## Architecture

**Claude thread** holds action cards and previews (session only — no cross-session memory).
**GDrive** holds the card tracker (persistent across runs). Gmail, Slack, and Jira are only
touched at the points shown below.

Between **triage** and **cards**, **Cowork** and **claude.ai** run doc lookup (grep or RAG)
plus a web search to confirm findings. The **Slack bot** does not use this step — it looks up
cards from the tracker and uses web search only.

### Core path

```mermaid
%%{init: {"flowchart": {"htmlLabels": true, "nodeSpacing": 48, "rankSpacing": 52, "padding": 16, "diagramPadding": 12}} }%%
flowchart LR
    subgraph triggers["Triggers"]
        direction TB
        T3["Slack"]
        T1["Cowork"]
        T2["claude.ai"]
    end

    subgraph storage["GDrive · persistent memory"]
        GD[("Card tracker")]
    end

    subgraph session["Claude thread · session only"]
        direction LR
        S1["① Load tracker"] --> S2["② Triage<br/>Gmail"] --> S2b["③ Research<br/>docs + web"] --> S3["④ Cards"] --> S4["⑤ Write tracker"] --> S5{"⑥ Next step"}
    end

    subgraph outputs["Outputs"]
        direction TB
        P1["Post to alerts"]
        P2["Draft · Gmail"]
        P3["Escalate · Jira · Slack"]
    end

    T3 --> S1
    T1 --> S1
    T2 --> S1

    T1 -.->|grep| S2b
    T2 -.->|RAG| S2b

    GD -->|read| S1
    S4 -->|write| GD

    S5 -->|scheduled| P1
    S5 -->|manual| P2
    S5 -->|manual| P3

    style session fill:none,stroke:#888,stroke-width:2px
    style storage fill:none,stroke:#666,stroke-width:1px
    style outputs fill:none,stroke:none
    style GD fill:#d4e8fc,stroke:#333
```

> **Note:** `Post to alerts` = your `#alerts` Slack channel. GitHub's Mermaid renderer treats `#` inside labels as a comment, so the channel name is spelled out in prose here and in the table below.

*③ Research* applies to **Cowork** (grep) and **claude.ai** (RAG) only. The **Slack bot** skips
triage and doc lookup — it reads the Slack post, looks up the card tracker, and uses web search
when product lookup is needed.

### Scheduled vs manual

| Step | Scheduled (Cowork) | Manual · claude.ai | Manual · Slack bot |
|---|---|---|---|
| ① Load tracker | Read card tracker from GDrive | Same | Lookup card from tracker |
| ② Triage | Inbox Probe — new cards or refresh from tracker | Inbox Probe · Email Q&A · Action Review | **Skipped** — Slack post is the source |
| ③ Research | grep over local docs → web search | RAG over Project knowledge → web search | Web search only (no doc access) |
| ④ Cards | Rendered in Claude thread only | Same | Respond from Slack context |
| ⑤ Write tracker | Written to GDrive when cards update | Same | Written on escalate |
| ⑥ Next step | Post summary to `#alerts` (always) | Draft email and/or Escalate (optional) | Respond · Escalate |

### Connector touchpoints

| System | When | What happens |
|---|---|---|
| **Product docs** | Step ③ · Cowork / claude.ai | grep (Cowork) or RAG (claude.ai), then web search |
| **GDrive** | Start + after cards | Load tracker; write back on card update |
| **Gmail** | Triage + manual draft | Read inbox threads; create draft replies (`threadId`, never sent) |
| **Slack** | Scheduled end + manual escalate | `#alerts` summary (scheduled); account-channel posts (after confirm) |
| **Jira** | Manual escalate only | Create/update tickets after you confirm preview |
| **Claude thread** | Steps ①–⑥ | Cards and previews — session only |

**Also required:** Chrome connector on Claude Desktop (posts scheduled alerts via browser).

---

## Before you start

### 1. Get access to Claude

You need a **Claude Team or Pro plan** with sufficient token limits for inbox-probe runs.
Sign up or ask your admin at [claude.ai](https://claude.ai).

### 2. Download Claude Desktop

Download from [claude.ai/download](https://claude.ai/download). This is used for scheduled
runs — Claude Desktop's Cowork feature supports scheduling.

### 3. Connect four connectors

In Claude → Settings → Connectors, connect all four. Do the same in Claude Desktop → Customize → Connectors.

| Connector | Used for | Permissions |
|---|---|---|
| **Gmail** | Read inbox, create draft replies | Read + create drafts (no send, no delete) |
| **Google Drive** | Store/read the card tracker | Read + create files (no delete, no org-wide) |
| **Slack** | Post alerts + draft account-channel messages | Read + post/draft + lookup (no admin) |
| **Jira (Atlassian)** | Create/update escalation tickets | Read + create/edit issues (no admin) |

Leave permissions at the default settings the connector prompts suggest.

### 4. Connect Chrome on Claude Desktop

In Claude Desktop → Customize → Connectors, connect "Chrome" under Desktop. This lets CSM Radar
post Slack alerts as a browser action, which is more reliable than the Slack MCP bot token for
scheduled posts.

### 5. Enable the Claude Slack app + create an alerts channel

1. Enable the Claude app (bot) in your Slack workspace. Your Slack admin may need to approve it.
2. Create a **private channel** and invite **only the Claude app bot** (no human members).
   This is your alerts channel (e.g. `#csm-radar-alerts`). Scheduled summaries post here.

Note the channel name — you'll need it during setup.

### 6. Prepare your account details

Before running the wizard, gather:

- **Client/account names** and their product subscriptions
- **IDs per account, per environment (PRD and DEV):** Org ID, Project ID, App ID; secondary
  product (e.g. CDP) environment IDs; chat/bot product IDs
- **One internal Slack channel per account** (your team's account-specific channel)
- **Your working timezone** (e.g. UTC+9 for KST, UTC+8 for SGT)
- (Optional) **Account tracking spreadsheet URLs** if you keep per-account sheets

### 7. Prepare your product documentation (recommended)

CSM Radar verifies every product claim against your documentation. Without docs, it falls back
to web search — slower and less precise.

**How to get your docs into Markdown:**
Scrape your official documentation website into Markdown files. Use Claude Code or another AI
agent tool to crawl your docs site and output one `.md` file per major section. Re-run when
your docs change significantly.

**How to load them:**

- **Web (claude.ai):** add the Markdown files to your Claude Project knowledge (see Step 4 below).
- **Cowork (Claude Desktop):** place the files in a path-accessible location; update the
  `references/doc-lookup.md` filenames in the configured `csm-radar` skill if needed.

---

## Setup (step by step)

### Step 1 — Install the setup wizard

1. Download or clone this repo.
2. In Claude → Settings → Skills, upload the `sova-setup/` folder as a skill.
3. Start a new chat and say: **"Hey Claude — I just added the sova-setup skill."**

Claude drives the rest. It confirms your connectors, fetches your Jira cloud ID automatically,
creates your Drive folder, finds your Slack IDs, and builds your account roster from whatever
you paste or share. At the end it hands you a configured `csm-radar` zip with no placeholders.

You can delete the `sova-setup` skill afterwards.

### Step 2 — Install the configured csm-radar skill

Upload the `csm-radar` zip from the wizard in Claude → Settings → Skills. Do this for both
your web (claude.ai) install and your Claude Desktop install.

### Step 3 — Create a Claude Project

Create a Project in claude.ai. The system prompt can be minimal — the skill contains all the
instructions. For example:

```
You are a CSM assistant. Load the csm-radar skill and run in Inbox Probe mode
against [your-email@company.com]. Do not stop early.
```

> You need **two separate Projects**: one for web (interactive use) and one for Cowork
> (scheduled runs). Upload your product docs to each Project's knowledge separately.

### Step 4 — Upload product docs to both Projects

Upload your scraped Markdown documentation to **both** the web Project and the Cowork Project.
The two surfaces do not share files.

**Web (claude.ai)** — add docs via Project → Files:

![Claude web Project with product docs in the Files panel](docs/images/image3.png)

**Cowork (Claude Desktop)** — add docs when creating the Project or via Project → Context:

![Cowork — Start a new project with docs attached](docs/images/image4.png)

![Cowork Project with Instructions, Scheduled tasks, and Context files](docs/images/image2.png)

### Step 5 — Set up scheduled runs (Claude Desktop)

Open Claude Desktop → your Cowork Project and tell Claude when to run. For example:

> "Set up a scheduled inbox probe every 4 hours on weekdays between 9 AM and 6 PM [your timezone]."

The Scheduled panel in your Cowork Project will show the recurring task once configured.

---

## Daily use

- **Scheduled run** → summary lands in your alerts channel. Skim the cards.
- **Claude Desktop** runs in the background. A spare machine works well for always-on scheduling.
- **In Slack** — @mention the Claude bot. Start fresh conversations with `/csm-radar` so the bot
  loads the skill and fetches the tracker before responding. The bot only remembers the current
  thread — not your claude.ai or Cowork sessions.
- **On the go** — reply to the Slack summary, or @mention the bot + `/csm-radar` to start fresh.
- **At the claude.ai console:**
  - `"follow up on card 3"` — refreshes a specific card
  - `"draft a reply"` — generates a draft (you review before sending)
  - `"create Jira + share to Slack"` → preview → confirm → done
  - `"mark card 4 resolved"` — keeps tomorrow's probe clean
  - `"ask Claude to update memory"` — fine-tunes triage (e.g. "exclude automated quota alerts",
    "exclude internal tooling update emails")

Gmail is read-only. All Jira and Slack actions need your confirmation. Your data stays in your
own connected accounts.

---

## Configuration reference

All `{{PLACEHOLDERS}}` in the skill files, filled by the wizard:

| Placeholder | File(s) | How to find |
|---|---|---|
| `{{YOUR_JIRA_CLOUD_ID}}` | `escalate.md`, `card-tracker.md` | `Atlassian:getAccessibleAtlassianResources` |
| `{{YOUR_JIRA_SITE}}` | `escalate.md`, `card-tracker.md` | Same tool — the `url` field |
| `{{CARD_TRACKER_FOLDER_ID}}` | `card-tracker.md`, `email-qna.md`, `action-review.md`, `escalate.md` | Google Drive folder URL → segment after `/folders/` |
| `{{YOUR_SLACK_WORKSPACE_ID}}` | `alerts-post.md` | `T…` in `app.slack.com/client/T…/C…` |
| `{{YOUR_ALERTS_CHANNEL_ID}}` | `alerts-post.md` | `C…` in the same URL |
| `{{YOUR_ALERTS_CHANNEL_NAME}}` | `alerts-post.md` | Channel name without `#` |
| `{{YOUR_TIMEZONE}}` / `{{YOUR_UTC_OFFSET}}` | `profile.md`, `alerts-post.md` | Your working timezone |
| `{{YOUR_JIRA_PROJECT_A}}` / `{{YOUR_JIRA_PROJECT_B}}` | `escalate.md` | `getVisibleJiraProjects` |
| `{{YOUR_FEATURE_REQUEST_FIELD_1}}` / `{{YOUR_FEATURE_REQUEST_FIELD_2}}` | `ticket-template.md` | `getJiraIssueTypeMetaWithFields` |

The account files (`client-ids.md`, `slack-channels.md`, `profile.md`) are rebuilt from your
roster by the wizard — they don't use token substitution, they're written fresh.

---

## Troubleshooting

**Probe crashes or cards are empty**
→ Check Gmail and Google Drive connectors. Verify `{{CARD_TRACKER_FOLDER_ID}}` is correct in `card-tracker.md`.

**Jira ticket creation fails**
→ Verify the Atlassian connector is active and `{{YOUR_JIRA_CLOUD_ID}}` is set in `escalate.md`.

**Slack summary not posting on scheduled runs**
→ Verify the Chrome connector is connected in Claude Desktop. Check workspace and channel IDs in `alerts-post.md`.

**Slack bot doesn't know about my previous conversations**
→ Expected — the bot has no cross-session memory. Always start with `/csm-radar` to load the skill and fetch the tracker.

**Product claims are wrong or outdated**
→ Re-scrape your documentation and replace the doc files in your Project knowledge (web) and Cowork Context (desktop).

**Cowork and web give different answers on the same question**
→ Expected — Cowork uses keyword search over local files; web uses RAG over Project knowledge. Keep both doc sets in sync.

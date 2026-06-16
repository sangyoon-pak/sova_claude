# CSM Radar

**A Claude-powered inbox copilot for enterprise Customer Success Managers.**

Automates the repetitive mechanics of CSM work — inbox triage, customer-email research,
Jira ticket creation, and Slack escalation — so your time goes to the customer, not to
copying context between tools.

## Demo

<video src="sova.mp4" controls width="100%"></video>

*[Download video](./sova.mp4)* if the player does not appear in your viewer.

---

## How it works

You install one skill (`sova-setup`) and talk to Claude. The wizard configures and packages
a second skill (`csm-radar`) tailored to your company, Jira setup, and accounts. You then
install the configured skill and use it every day.

```
You (once)           You (daily)
────────────         ────────────────────────────────────────────
Install              "Scan my inbox"  →  Action cards
sova-setup      →   "How do I answer this?" →  Talking points + draft
     ↓               "Create a Jira ticket" →  Preview → Confirm → Done
Claude produces      Scheduled run  →  Slack summary in #alerts-channel
csm-radar.zip
     ↓
Install
csm-radar
```

---

## Modes

| Mode | How to trigger | What happens |
|---|---|---|
| **Inbox Probe** | "Scan my inbox" / scheduled | Reads Gmail (read-only), triages threads, renders interactive action cards |
| **Email Q&A** | Paste an email + "how do I answer this?" | Researches your product docs + web, returns grounded answer + optional draft |
| **Action Review** | "Follow up on the Acme thread" | Restores thread history from memory, refreshes the card |
| **Escalate** | Card button or direct ask | Builds Jira ticket or Slack post → preview → you confirm → done |

All Jira and Slack actions are **preview-then-confirm**. Nothing is filed silently.

---

## Setup

Read **[SETUP-GUIDE.md](./SETUP-GUIDE.md)** for the full walkthrough.

The short version:

1. **Connect four connectors** in Claude (Settings → Connectors): Gmail · Google Drive · Slack · Jira (Atlassian).
2. **Install `sova-setup`** as a Claude skill (Settings → Skills → upload the `sova-setup/` folder).
3. **Start a chat** and say: `"Hey Claude — I just added the sova-setup skill."`
4. Claude walks you through everything and hands back a configured `csm-radar` zip.
5. **Install `csm-radar`**, create a Claude Project, add your product docs, and try: `"Scan my inbox."`

---

## Repo structure

```
csm-radar/
├── README.md
├── SETUP-GUIDE.md          ← read this before setup
└── sova-setup/             ← install this skill first; it builds csm-radar for you
    ├── SKILL.md            ← wizard entry point
    ├── references/
    │   ├── prerequisites.md      ← what to prepare before running the wizard
    │   ├── field-reference.md    ← what values the wizard collects and how
    │   └── fill-and-package.md   ← how the wizard fills placeholders and packages the skill
    └── assets/
        └── csm-radar-template/   ← the template the wizard configures ({{PLACEHOLDERS}})
            ├── SKILL.md.tmpl
            ├── references/       ← inbox-probe, escalate, card-tracker, etc.
            └── assets/
                └── ticket-template.md
```

Users never touch the template files directly — the wizard handles everything.

---

## Requirements

- **Claude Team or Pro plan** (for token budget on inbox-probe runs)
- **Claude Desktop** (for scheduled runs via Cowork)
- **Four MCP connectors:** Gmail · Google Drive · Slack · Jira (Atlassian)
- **Chrome connector** on Claude Desktop (for browser-based Slack posting on scheduled runs)

---

## Privacy

- Gmail is **read-only** — CSM Radar never sends or schedules email.
- All Jira and Slack actions require your explicit confirmation.
- Your data stays in your own connected accounts.

---

## License

MIT — see [LICENSE](./LICENSE).

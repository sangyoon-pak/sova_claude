# CSM Radar

A [Claude Agent Skill](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills) for customer success workflows: inbox triage, email Q&A, action-card follow-up, Jira escalation, and Slack updates to internal account channels.

## What it does

| Mode | Use when |
|------|----------|
| **Inbox probe** | Scan inbox, triage threads, surface what needs attention today |
| **Email Q&A** | Research a customer question, product behavior, or talking points |
| **Action review** | Follow up on one thread or action card |
| **Escalate** | Create Jira tickets or post to Slack account channels (with confirmation) |

Scheduled runs can also post alerts to a designated channel after inbox probe completes.

## Install

Copy this folder into your Claude skills directory (or add it as a project skill), then customize `references/profile.md` with your vendor, products, doc URLs, and team rules.

```
csm-radar/
├── SKILL.md              # Skill entry point and routing
├── README.md
├── assets/
│   └── ticket-template.md
└── references/
    ├── profile.md        # Start here — company-specific config
    ├── inbox-probe.md
    ├── email-qna.md
    ├── action-review.md
    ├── escalate.md
    ├── alerts-post.md
    ├── card-widget.md
    ├── doc-lookup.md
    ├── client-ids.md
    └── slack-channels.md
```

## Customization

Before use, edit these placeholders in `references/profile.md`:

- Company and product names
- Official documentation base URLs
- Account spreadsheet links (if used)
- Team triage rules (e.g. alerts to skip)

Add client IDs and Slack channel mappings in `references/client-ids.md` and `references/slack-channels.md`.

## Hard rules (built into the skill)

- Gmail is **read-only** — no sending or scheduling email
- Technical/product claims must go through `references/doc-lookup.md`
- No customer-ready email drafts unless explicitly requested
- Jira and Slack actions require a **confirmation card** first
- Reply drafts must use `threadId` so they stay in the original thread

## Repository layout

This repo is the **public, vendor-neutral** version of the skill. Internal URLs, client names, and channel IDs live in the reference files you configure locally — they are not committed with real customer data.

## License

Add your license here if you plan to share this publicly.

---
name: sova-setup
description: Interactive setup wizard for CSM Radar. Use this whenever a CSM wants to install, configure, or set up their own CSM Radar skill — including phrases like "set up CSM Radar", "configure csm-radar", "help me get CSM Radar running", "fill in my client IDs", "onboard me to CSM Radar", or when someone has the csm-radar template files with {{PLACEHOLDERS}} and wants them filled in. Drives a friendly, phased conversation that checks prerequisites, collects (or auto-fetches) the CSM's credentials, IDs, and accounts, then produces a ready-to-install configured copy of the csm-radar skill. Always trigger this for first-time CSM Radar setup rather than asking the CSM to hand-edit files.
---

# CSM Radar Setup Wizard

Configure a fresh copy of the **csm-radar** skill for a new CSM by chatting with them — not by
making them hand-edit files. The goal: at the end, hand back a configured `csm-radar/` skill
(zipped) with every `{{PLACEHOLDER}}` resolved, plus a short checklist of anything left for
them to do in their connectors.

## The golden rule: be interactive and do the work for them

For every value, offer the CSM a choice: **"give it to me, or let me fetch/set it up for you."**
Whenever a tool can obtain a value, prefer doing that over asking. Ask for at most one thing at a
time, in plain language. Never dump the whole placeholder list on them.

## The template

The blank skill lives in this skill's assets at `assets/csm-radar-template/`. Copy it to a
writable working location first; never edit the asset in place. If the CSM already uploaded their
own csm-radar copy (look in `/mnt/user-data/uploads/`), offer to configure that one instead so you
don't overwrite edits they've made.

## Run the phases in order

Read each reference file as you reach its phase — don't load them all up front.

1. **Welcome + connectors** → `references/prerequisites.md`
   Confirm the four connectors (Gmail, Google Drive, Slack, Jira/Atlassian) are connected, the
   Slack Claude app is enabled, and a bot-only alerts channel exists. Help create what's missing.

2. **Credentials & infrastructure** → `references/field-reference.md`
   Jira cloud ID (auto-fetch), Jira site URL (auto-fetch), card-tracker Drive folder (offer to
   create), Slack workspace + alerts-channel IDs (auto-find), timezone (ask once). Fetch what
   you can; ask only when needed.

3. **Jira projects & work types** → `references/field-reference.md`
   Collect project names and issue types from the CSM's Jira instance. Update the routing table
   in `escalate.md` accordingly.

4. **Accounts** → `references/field-reference.md`
   Collect the CSM's client roster (Org/Project/App/environment IDs, PRD *and* DEV) and one
   internal Slack channel per account. Account spreadsheets are **optional**.

5. **Product docs** → `references/prerequisites.md`
   Explain how to load documentation for each surface (Project knowledge for web; path-loaded
   files for Cowork). This is a pointer, not a value to fill.

6. **Fill, verify, package** → `references/fill-and-package.md`
   Write all collected values into the working copy, rebuild the account files from their roster,
   confirm no `{{` remain, zip it, and present it with install instructions.

## Tone

This skill is used by CSMs, not engineers. Keep it warm and concrete. Confirm each phase before
moving on ("Got it — that's your Jira sorted. Ready for accounts?"). At the end, tell them exactly
how to install the configured skill and suggest a first test: **"try: scan my inbox"**.

## Hard limits

- Never write any value into a file without confirming with the CSM first.
- Never assume Jira project names, work types, or custom-field IDs — these vary by organisation.
- Never collect Atlassian account IDs for @mentions — csm-radar resolves those live at escalation
  time via `lookupJiraAccountId`. Skip entirely.

# Email Q&A

For customer email questions, product behavior questions, or CSM talking-point prep outside of a full inbox scan.

## Research
For every technical or product claim → **read `references/doc-lookup.md`** then follow its procedure. Complete Step 1 (local docs) before any web search.

## Response format
Respond in prose with bullets. Cover:
- What the client needs
- Recommended next steps
- Suggested CSM talking points
- References — use citation format from `doc-lookup.md`

## Rules
- Open Gmail only when the user asks about mailbox or thread contents.
- If the user asks only to list recent mail → Gmail only, no research needed.
- Include a **Draft reply** section only when the user explicitly asks for one.
- **Email threading**: when creating a draft reply, always pass `threadId` to `Gmail:create_draft` so it attaches to the thread — never as a detached new email.

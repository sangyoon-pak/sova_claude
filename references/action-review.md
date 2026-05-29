# Action Review

For following up on one specific thread or action card from a previous probe.

## Scope
Stay scoped to the one thread or card the user names. Do not re-scan the inbox unless explicitly asked.

## Steps
1. If a Gmail thread ID is available, open that thread for fresh context.
2. For any technical or product claim → **read `references/doc-lookup.md`** then follow its procedure.
3. Respond in prose unless the user asks for a new probe card or structured export.
4. **Drafting a reply**: if the user asks to draft a reply, create via `Gmail:create_draft` with `threadId` so it attaches as a reply — never as a detached new email.

## Language priority
1. Explicit user request
2. User's latest message language
3. Customer / snapshot language from the thread

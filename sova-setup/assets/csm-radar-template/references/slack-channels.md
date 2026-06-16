# Slack Account Channels

Internal account channels — used when posting updates or escalations to the right client channel.

> ⚙️ **SETUP — THIS FILE IS A TEMPLATE.**
> Replace the rows below with the internal Slack channels for your own accounts.
> One row per account. Delete the sample rows once you've added your own.

## Channel list

| Client | Slack Channel Name | Notes |
|---|---|---|
| {{CLIENT_A_NAME}} | `{{client_a_channel_name}}` | # public |
| {{CLIENT_B_NAME}} | `{{client_b_channel_name}}` | 🔒 private |
| {{CLIENT_C_NAME}} | `{{client_c_channel_name}}` | 🔒 private |

## Channel lookup rules

- Match by client name keywords (e.g. "{{CLIENT_A_NAME}}" → `{{client_a_channel_name}}`)
- If a client spans regions, also match by country/region code in the request
- If the client maps to multiple channels and it is ambiguous, ask the CSM to confirm which one
- If the client is NOT in this list, tell the CSM and ask them to provide the channel name manually

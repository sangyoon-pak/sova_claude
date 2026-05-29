# Slack Account Channels

Internal account channels — used when posting updates or escalations to the right client channel.

## Channel list

| Client | Slack Channel Name | Notes |
|---|---|---|
| [CLIENT_A] | `client_[region]_[client_a]` | 🔒 private |
| [CLIENT_B] | `client_[region]_[client_b]` | # public |
| [CLIENT_C] | `client_[region]_[client_c]` | 🔒 private |
| [CLIENT_D] | `client_[region]_[client_d]` | 🔒 private |

> **How to populate:** Add one row per client. Use a consistent naming convention such as `client_[country/region]_[clientname]`. Mark public (`#`) or private (`🔒`) channels clearly.

## Channel lookup rules

- Match by client name keywords (e.g. "ClientA" → `client_xx_clienta`)
- For clients with multiple regions, also match by country/region code in the request (e.g. SG, US, APAC)
- If the client maps to multiple channels (unlikely), ask the CSM to confirm which one
- If the client is NOT in this list, tell the CSM and ask them to provide the channel name manually

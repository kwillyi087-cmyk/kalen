# Project notes

## Email

Use the **Atomic Mail** MCP server for all email — sending, reading, searching, and
replying. The inbox is:

```
Kalen.Wilson@atomicmail.ai
```

Do **not** use the Gmail or Microsoft 365 / Outlook connectors for email in this
project, even when they are connected and available. Atomic Mail is the mailbox.

### Server

| | |
|---|---|
| Name | `atomic-mail` |
| URL | `https://mcp.atomicmail.ai/mcp` |
| Transport | HTTP |
| Auth | `Authorization: Bearer <inbox API key>` |
| Scope | user (`~/.claude.json`) |

The API key is **not** stored in this repository. It lives in the user-scope Claude
config so it stays out of version control. To recreate the server on a new machine,
get the key from the inbox's Connect dialog in the Atomic Mail dashboard and run:

```bash
claude mcp add --scope user --transport http atomic-mail \
  https://mcp.atomicmail.ai/mcp \
  --header "Authorization: Bearer <inbox API key>"
```

Verify with `claude mcp list` — it should report `atomic-mail ... Connected`.

### Tools

`read_inbox`, `read_message`, `search_messages`, `send_email`, `reply_to_message`,
`list_agents`, `search`, `fetch`, `run_preset`, `jmap_request`, `help`.

Every tool takes an optional `agent_id`. The key authenticates exactly one inbox
(`Kalen.Wilson`), so **omit `agent_id`** and it defaults to the right mailbox.

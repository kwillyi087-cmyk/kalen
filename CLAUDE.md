# Project notes

## Email

Use the **Atomic Mail** MCP server for all email — sending, reading, searching, and
replying. The inbox is:

```
Kalen.Wilson@atomicmail.ai
```

Do **not** use the Gmail or Microsoft 365 / Outlook connectors for email in this
project, even when they are connected and available. Atomic Mail is the mailbox.

### Known server bugs and limits

Read this before composing anything. Two quirks of the live server, both found the
hard way. Neither is documented by Atomic Mail, and neither is caused by a
malformed client request.

**1. `send_email` bodies must be single-line ASCII.**

A body containing a raw newline or a non-ASCII character (em dash, curly quote)
fails with:

```
send_email error: send_email is not valid JSON:
Bad control character in string literal at position <n>
```

The request is valid JSON going out — newlines correctly escaped as `\n` — so
this is the server mangling its own re-serialization of the arguments, writing
raw control characters where it should escape them. The giveaway: `<n>` indexes
the whole arguments blob, and shifts by exactly one character when the recipient
address gets one character longer.

Consequence: **multi-paragraph emails are not currently possible.** Compose the
body as one flowing paragraph of plain ASCII. Don't retry a multi-line body
expecting a different result — every attempt fails identically.

**2. Rate limiting counts connection attempts, not sends.**

```
{"error":"rate_limited",
 "error_description":"Too many connection attempts for this API key.
                      Wait about a minute and retry."}
```

The limit is per API key and applies to *all* tools — a throttled key fails
`read_inbox` and `help` too, not just `send_email`. Each MCP call opens a fresh
session and counts against it, so a burst of calls (including retries and
diagnostic probes) trips it easily. Space calls out by roughly a minute and let
the limit clear fully before retrying; hammering it keeps it closed and masks
whatever the real error was.

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

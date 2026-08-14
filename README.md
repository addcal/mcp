# AddCal MCP Server

[![smithery badge](https://smithery.ai/badge/hello-ub6m/addcal-calendar-links)](https://smithery.ai/servers/hello-ub6m/addcal-calendar-links)
[![MCP Registry](https://img.shields.io/badge/MCP%20Registry-co.addcal%2Faddcal-blue)](https://registry.modelcontextprotocol.io/v0/servers?search=addcal)

A remote [Model Context Protocol](https://modelcontextprotocol.io) server for [AddCal](https://addcal.co). Connect Claude, ChatGPT, Cursor or any MCP client and manage **the calendars other people subscribe to**.

```
https://addcal.co/mcp
```

No install, no API key. It is a hosted streamable HTTP server and authentication is OAuth.

## This is not a Google Calendar MCP

Almost every calendar MCP server does the same job: it reads and writes *your own* schedule. Useful, and well covered already.

AddCal is the other side of the calendar. It manages the events you **publish** for an audience: a class timetable, a fixture list, a release schedule, a conference programme. The people receiving those events never get an invite. They subscribe to a feed, or they click an add-to-calendar link, and the event lands in whichever calendar app they already use.

That difference shows up in the tools. There is no `accept-invite`, because nobody is invited. There is `list-subscribers`, `get-analytics` and `create-calendar-collection`, because the questions you ask about a published calendar are how many people follow it and whether anyone actually added the event.

Use a Google Calendar MCP to run your day. Use this one to run a calendar other people follow.

## Quick start

Pick your client. Cursor and VS Code install in one click.

[![Add to Cursor](https://img.shields.io/badge/Add%20to-Cursor-000000?logo=cursor)](https://cursor.com/en/install-mcp?name=addcal&config=eyJ1cmwiOiJodHRwczovL2FkZGNhbC5jby9tY3AifQ%3D%3D)
[![Add to VS Code](https://img.shields.io/badge/Add%20to-VS%20Code-007ACC?logo=visualstudiocode)](vscode:mcp/install?%7B%22name%22%3A%22addcal%22%2C%22type%22%3A%22http%22%2C%22url%22%3A%22https%3A%2F%2Faddcal.co%2Fmcp%22%7D)

**Claude Code**

```bash
claude mcp add --transport http addcal https://addcal.co/mcp
```

Then run `/mcp` inside Claude Code to sign in.

**Claude Desktop and claude.ai**

Settings, Connectors, Add custom connector, then paste `https://addcal.co/mcp`. Custom connectors need a paid plan.

**ChatGPT**

Settings, Apps & Connectors, Advanced, turn on Developer mode, then add `https://addcal.co/mcp` as a connector. Requires a paid plan on the web app.

**Anything else**

Any client that speaks streamable HTTP with OAuth. Windsurf, Goose, Zed and the JetBrains AI Assistant all take a remote MCP URL.

```json
{
  "mcpServers": {
    "addcal": {
      "type": "http",
      "url": "https://addcal.co/mcp"
    }
  }
}
```

## Authentication

OAuth 2.1 with PKCE and dynamic client registration. There is no API key to copy and paste.

Your client registers itself, you approve it in a browser, and it receives a token scoped to your team. It can reach exactly what you can reach, no more. Revoke any connected client from [Settings, API](https://addcal.co/user/api-tokens).

## What you can ask for

```
What have I got on next week?
Create a team standup every weekday at 9am and show me the dates first.
Who has RSVPed yes to the launch party?
Duplicate last month's workshop into March.
Which of my events got the most add-to-calendar clicks?
```

## Tools

19 tools. `RO` is read only. Required parameters are marked `*`.

### Search

| Tool | | Parameters |
|---|---|---|
| `search` | RO | `query*`, `limit`, `team_id` |

One call across calendars, collections, events and RSVPs, returning the ids every other tool needs. Start here whenever something is named rather than identified.

### Calendars

| Tool | | Parameters |
|---|---|---|
| `list-calendars` | RO | `search`, `limit`, `team_id` |
| `create-calendar` | | `name*`, `description`, `timezone`, `team_id` |

### Collections

A collection bundles several calendars behind one public page and subscribe link.

| Tool | | Parameters |
|---|---|---|
| `list-calendar-collections` | RO | `team_id` |
| `create-calendar-collection` | | `name*`, `calendar_ids*`, `description`, `allow_subscriber_selection`, `team_id` |

### Subscribers

| Tool | | Parameters |
|---|---|---|
| `list-subscribers` | RO | `calendar_id*`, `status`, `search`, `limit` |

Returns personal data. Names and emails exist only for calendars that ask subscribers for details, so many rows are anonymous.

### Events

| Tool | | Parameters |
|---|---|---|
| `list-events` | RO | `calendar_id`, `search`, `range`, `start_date`, `end_date`, `limit`, `team_id` |
| `get-event` | RO | `event_id*` |
| `create-event` | | `calendar_id*`, `title*`, `date_start*`, `date_end*`, `timezone`, `description`, `location`, `is_all_day`, `is_draft`, `has_rsvp`, `recurrence_rule`, `series_title`, `reminder_before` |
| `update-event` | | `event_id*`, `title`, `description`, `location`, `date_start`, `date_end`, `timezone`, `is_all_day`, `is_draft`, `has_rsvp`, `recurrence_rule`, `reminder_before` |
| `duplicate-event` | | `event_id*`, `target_calendar_id`, `title`, `date_start`, `date_end` |
| `delete-event` | **destructive** | `event_id*`, `confirm*`, `notify_invitees` |
| `preview-recurring-dates` | RO | `recurrence_rule*`, `date_start*`, `date_end*`, `timezone`, `is_all_day`, `count` |

Recurrence uses RFC 5545 rules without the `RRULE:` prefix, such as `FREQ=WEEKLY;BYDAY=MO;COUNT=10`. Run `preview-recurring-dates` before creating a series to check the dates land where you expect.

### RSVPs

| Tool | | Parameters |
|---|---|---|
| `list-rsvps` | RO | `event_id*`, `response_type`, `search`, `limit` |
| `create-rsvp` | **destructive** | `event_id*`, `name*`, `email*`, `response_type*`, `confirm*`, `custom_fields`, `send_confirmation` |

`create-rsvp` emails the attendee by default. Set `send_confirmation` to false to record an RSVP silently.

### Invites

| Tool | | Parameters |
|---|---|---|
| `list-invites` | RO | `event_id*`, `status`, `search`, `limit` |
| `send-event-invite` | **destructive** | `event_id*`, `email*`, `confirm*` |

### Account

| Tool | | Parameters |
|---|---|---|
| `get-analytics` | RO | `calendar_id`, `event_id`, `start_date`, `end_date`, `timezone`, `unique_only`, `include_series`, `team_id` |
| `get-team-usage` | RO | `team_id` |

## Destructive tools

Three tools are annotated destructive, and all three require an explicit `confirm` parameter:

- `send-event-invite` and `create-rsvp` email a real person. That cannot be recalled.
- `delete-event` is permanent, and deleting a recurring event removes every instance in the series.

A well-behaved client surfaces these for approval before running them. Check the name and address when your assistant asks.

## Plans and limits

The server is included on every plan, and the tools respect whatever your plan already allows. Collections, calendar invites and analytics stay gated to the plans that include them. When a tool hits a limit it explains why rather than retrying.

## Links

- [MCP server guide](https://addcal.co/documentation/mcp-server)
- [Full tool reference](https://addcal.co/docs/mcp)
- [REST API reference](https://addcal.co/docs)
- [Smithery listing](https://smithery.ai/servers/hello-ub6m/addcal-calendar-links)
- [Official MCP registry entry](https://registry.modelcontextprotocol.io/v0/servers?search=addcal)

## About this repository

AddCal's MCP server is hosted, so there is nothing here to install or run. This repository holds the published [`server.json`](server.json) manifest and the documentation for the remote endpoint. The implementation lives in the main AddCal application.

Found a problem with a tool, or want one that does not exist yet? [Open an issue](https://github.com/addcal/mcp/issues).

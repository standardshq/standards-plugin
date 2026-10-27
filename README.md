# Standards plugin for Claude

[Standards](https://standards.new) is a business workspace — CRM, documents, drives, email, forms and AI agents in one app — whose data model each team declares for itself: contacts and deals, incidents and suppliers, or whatever the business runs on. This plugin connects Claude Code and Claude Cowork to your workspace, so Claude can search, count, create and update your records, read and file the documents attached to them, and use the workspace's memory, always with the rights of your role.

## Install

```bash
/plugin marketplace add standardshq/standards-plugin
/plugin install standards@standards
```

The first tool call opens the Standards consent screen in your browser: sign in, pick the workspace, and Claude is connected. No API key to paste. To connect with an API key instead, see [docs.standards.new/docs/mcp](https://docs.standards.new/docs/mcp).

## What you can ask

- "Which deals in negotiation haven't moved in 30 days? Group them by owner."
- "Create a follow-up task for each contact I met at yesterday's event."
- "Summarise the latest contract attached to ACME and tell me when it renews."
- "From now on, call clients 'members' in your answers." (proposed to the workspace memory)

## What's inside

| Path | What it is |
|---|---|
| `.mcp.json` | The `standards` MCP server, `https://api.standards.new/v1/mcp` (streamable HTTP, OAuth) |
| `skills/standards/SKILL.md` | How Claude works in a Standards workspace: the eight tools and `<kind>:<id>` references, the schema as contract, exact vs ranked search, totals, safe writes, documents, memory |
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | Lets this repository act as its own marketplace |

## Permissions

Claude sees only the tools your role allows, narrowed by the access level you picked on the consent screen (Read, Write or Full, changeable later in Connected apps), and every call is checked again against the object it touches. Deletes are soft and restorable, and the skill asks Claude to confirm each one with you first.

Support: contact@standards.new · Docs: [docs.standards.new](https://docs.standards.new)

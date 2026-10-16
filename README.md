# standards — Claude Code plugin

Connects Claude Code (and Claude Cowork) to a [standards.new](https://standards.new) workspace through its remote MCP server, and ships a skill that teaches Claude how to use it.

## Install

```bash
/plugin marketplace add standardshq/standards-plugin
/plugin install standards@standards
```

The first tool call opens the standards.new consent screen in your browser (OAuth, no API key to paste). To connect with an API key instead, see [docs.standards.new/docs/mcp](https://docs.standards.new/docs/mcp).

## Contents

| Path | What it is |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest |
| `.claude-plugin/marketplace.json` | Lets this repository act as its own marketplace |
| `.mcp.json` | The `standards` MCP server (`https://api.standards.new/v1/mcp`, streamable HTTP) |
| `skills/standards/SKILL.md` | How to read and write a workspace safely through the tools |

Support: contact@standards.new

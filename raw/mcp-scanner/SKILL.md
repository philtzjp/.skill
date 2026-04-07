---
name: mcp-scanner
description: 接続中のMCPサーバーをスキャンし、サーバー/ツールごとのコンテキスト消費量を報告。コストの高いMCPの無効化・削除を提案する。/mcp-scannerで起動。
---

# MCP Scanner

The keywords "MUST", "NEVER", "SHOULD", and "MAY" in this document are to be interpreted as described in RFC 2119.

## Overview

Scans all connected MCP servers in the current session, estimates how much context window each server consumes, and presents an actionable report. Can disable or remove MCPs that are wasting context.

## Procedure

### 1. Inventory Connected MCP Servers

MUST gather MCP server information from two sources:

#### Source A: Active tools in this session

Scan the conversation context for tool definitions matching the `mcp__<server>__<tool>` pattern. Group tools by server name. For each tool, note:
- Tool name
- Whether it is **loaded** (full schema visible) or **deferred** (name only, listed in system-reminder)
- Description length (if loaded)

#### Source B: Configuration files

Read the following files (skip if not found):

```
~/.claude/settings.json          → enabledPlugins
~/.claude/settings.local.json    → enabledMcpjsonServers, enableAllProjectMcpServers, permissions.allow
.mcp.json                        → mcpServers (project-level)
```

Cross-reference: identify servers that are **configured but not connected** (config exists but no tools appeared) and servers that are **connected but not in config** (e.g., injected by plugins).

### 2. Estimate Context Consumption

For each MCP server, estimate token usage:

| Component | Estimation Method |
|-----------|------------------|
| **Loaded tool schemas** | Count characters in the tool's JSON schema definition ÷ 4 ≈ tokens |
| **Deferred tools** | ~20 tokens per tool name (name + type line in system-reminder) |
| **Server instructions** | Count characters in any `# MCP Server Instructions` block for that server ÷ 4 |
| **System-reminder overhead** | Fixed ~50 tokens per server for framing text |

Calculate per-server totals and a grand total across all MCP servers.

### 3. Present Report

MUST present the report in this format:

```
## MCP Scanner Report

| Server | Tools | Loaded | Deferred | Est. Tokens | Status |
|--------|-------|--------|----------|-------------|--------|
| se-piace | 3 | 1 | 2 | ~850 | ✅ active |
| repomix | 8 | 0 | 8 | ~320 | 💤 all deferred |
| playwright | 12 | 5 | 7 | ~2,400 | ✅ active |
| context7 | 2 | 2 | 0 | ~600 | ✅ active |

**Total MCP context: ~4,170 tokens** (≈ 0.4% of 1M context)

### Configured but not connected
- `console-ninja` (in settings.local.json)

### Recommendations
- `playwright` is consuming the most context (2,400 tokens) with 12 tools.
  Consider disabling if not needed for this session.
```

Adjust recommendations based on:
- Servers with many loaded (non-deferred) tools that aren't being used
- Servers consuming > 1,000 tokens
- Servers the user hasn't called any tools from in this session

### 4. Offer Actions

After presenting the report, MUST ask the user using AskUserQuestion:

```
What would you like to do?

1. No changes — keep everything as-is
2. Disable specific MCP server(s)
3. Remove specific MCP server(s) from config permanently
```

If the user chooses option 2 or 3, ask which server(s) to target.

### 5. Disable an MCP Server

To **disable** (reversible — server stays in config but won't load):

#### Plugin-based servers (in `~/.claude/settings.json`)

Set the plugin to `false` in `enabledPlugins`:

```json
{
  "enabledPlugins": {
    "playwright@claude-plugins-official": false
  }
}
```

#### Project-level servers (in `.mcp.json`)

Add `"disabled": true` to the server entry:

```json
{
  "mcpServers": {
    "some-server": {
      "command": "npx",
      "args": ["..."],
      "disabled": true
    }
  }
}
```

#### Selectively enabled servers (in `~/.claude/settings.local.json`)

Remove the server name from `enabledMcpjsonServers` array.

MUST confirm the change with the user before writing.

### 6. Remove an MCP Server

To **remove** (permanent — deletes config entry entirely):

- For `.mcp.json`: delete the server key from `mcpServers`
- For `~/.claude/settings.json`: delete the key from `enabledPlugins`
- For `~/.claude/settings.local.json`: remove from `enabledMcpjsonServers` and clean up related `permissions.allow` entries

MUST show a diff preview of the change and get explicit user confirmation before writing.

MUST remind the user: "Changes take effect on the next Claude Code session. Connected servers in the current session will remain active until restart."

## Prohibitions

- NEVER modify config files without explicit user confirmation
- NEVER remove or disable an MCP server the user didn't specifically select
- NEVER edit `~/.claude.json` (internal state file, not user config)
- NEVER guess token counts for tools you can't see — only count what's visible in the current context
- NEVER uninstall plugin packages — only toggle config. Users can uninstall plugins themselves via `/plugins`

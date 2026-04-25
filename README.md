# Zotero Helper

A guide for managing Zotero libraries using Claude Code with the [zotero-mcp-server](https://github.com/yourusername/zotero-mcp-server).

## Prerequisites

- [Zotero](https://www.zotero.org/) desktop app (v8+) running locally
- [Claude Code](https://claude.com/claude-code) CLI installed
- `zotero-mcp-server` installed via pipx
- A Zotero Web API key (get one at https://www.zotero.org/settings/keys)
- Your Zotero library ID (numeric, found on the same settings page)

## MCP Server Setup

### 1. Install zotero-mcp-server

```bash
pipx install zotero-mcp-server
```

> **Symlink conflict:** If you previously installed via `uv`, the symlink at
> `~/.local/bin/zotero-mcp` may point to the old install. Fix it:
> ```bash
> ln -sf ~/.local/pipx/venvs/zotero-mcp-server/bin/zotero-mcp ~/.local/bin/zotero-mcp
> ```

### 2. Configure the MCP server in Claude Code

Add to your **global** Claude Code config (`~/.claude.json`) under `mcpServers`:

```json
{
  "mcpServers": {
    "zotero": {
      "command": "/Users/<you>/.local/bin/zotero-mcp",
      "env": {
        "ZOTERO_LOCAL": "true",
        "ZOTERO_EMBEDDING_MODEL": "default",
        "ZOTERO_API_KEY": "<your-api-key>",
        "ZOTERO_LIBRARY_ID": "<your-numeric-user-id>"
      }
    }
  }
}
```

> **Important:** Without both `ZOTERO_API_KEY` and `ZOTERO_LIBRARY_ID`, the MCP
> server runs in **local-only mode** — all write operations will fail.

### 3. Auto-allow all Zotero MCP tools (optional)

Create `.claude/settings.json` in your project:

```json
{
  "permissions": {
    "allow": [
      "mcp__zotero__*"
    ]
  }
}
```

### 4. Reconnect after config changes

```
/mcp
```

## Notes

- The local Zotero API (`localhost:23119`) is **read-only**. Writes go through the Zotero Web API, which is what the MCP server uses in hybrid mode.
- See [AGENTS.md](AGENTS.md) for known workarounds when using the MCP tools.

# Zotero Helper

A guide for managing Zotero libraries using Claude Code with the [zotero-mcp-server](https://github.com/yourusername/zotero-mcp-server).

## Prerequisites

- [Zotero](https://www.zotero.org/) desktop app (v8+) running locally
- [Claude Code](https://claude.com/claude-code) CLI installed
- `zotero-mcp-server` installed via pipx:
  ```bash
  pipx install zotero-mcp-server
  ```
- A Zotero Web API key (get one at https://www.zotero.org/settings/keys)
- Your Zotero user/library ID (numeric, found on the same settings page)

## MCP Server Setup

### 1. Install zotero-mcp-server

```bash
pipx install zotero-mcp-server
```

Verify the installation:

```bash
zotero-mcp version
```

> **Symlink conflict:** If you previously installed via `uv`, the symlink at
> `~/.local/bin/zotero-mcp` may point to the old uv install. Fix it:
> ```bash
> ln -sf ~/.local/pipx/venvs/zotero-mcp-server/bin/zotero-mcp ~/.local/bin/zotero-mcp
> ```

### 2. Configure the MCP server in Claude Code

Add the server to your **global** Claude Code config (`~/.claude.json`) under `mcpServers`:

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

| Variable | Required | Description |
|---|---|---|
| `ZOTERO_LOCAL` | Yes | Set to `"true"` to read from the local Zotero instance |
| `ZOTERO_API_KEY` | Yes (for writes) | Web API key — enables hybrid mode (local reads + web writes) |
| `ZOTERO_LIBRARY_ID` | Yes (for writes) | Your numeric Zotero user ID |
| `ZOTERO_EMBEDDING_MODEL` | No | Set to `"default"` for semantic search support |

> **Important:** Without both `ZOTERO_API_KEY` and `ZOTERO_LIBRARY_ID`, the MCP
> server runs in **local-only mode** — all write operations will fail with
> "Cannot perform write operations in local-only mode."

### 3. Auto-allow all Zotero MCP tools (optional)

Create or edit `.claude/settings.local.json` in your project:

```json
{
  "permissions": {
    "allow": [
      "mcp__zotero__*"
    ]
  }
}
```

This prevents Claude Code from prompting on every Zotero MCP tool call.

### 4. Reconnect after config changes

After editing the config, reconnect the MCP server inside Claude Code:

```
/mcp
```

## Available MCP Tools

### Read Operations

| Tool | Description |
|---|---|
| `zotero_search_items` | Search by title, creator, year |
| `zotero_search_by_tag` | Filter items by tags |
| `zotero_advanced_search` | Multi-criteria search |
| `zotero_semantic_search` | AI-powered topic search |
| `zotero_get_item_metadata` | Get item details (markdown or BibTeX) |
| `zotero_get_item_fulltext` | Read full paper text |
| `zotero_get_item_children` | List attachments and notes |
| `zotero_get_collections` | List all collections |
| `zotero_get_collection_items` | List items in a collection |
| `zotero_get_tags` | List all tags |
| `zotero_get_annotations` | Get highlights and annotations |
| `zotero_get_notes` | Retrieve notes |
| `zotero_get_recent` | Recently added items |
| `zotero_list_libraries` | List accessible libraries |
| `zotero_list_feeds` | List RSS feed subscriptions |

### Write Operations (require API key + library ID)

| Tool | Description |
|---|---|
| `zotero_update_item` | Update item metadata (title, date, publisher, tags, etc.) |
| `zotero_manage_collections` | Add/remove items from collections |
| `zotero_create_collection` | Create new collections or subcollections |
| `zotero_batch_update_tags` | Batch add/remove tags across items |
| `zotero_merge_duplicates` | Merge duplicate items (dry-run supported) |
| `zotero_find_duplicates` | Find duplicates by title/DOI |
| `zotero_add_by_doi` | Add paper by DOI with auto-fetched metadata |
| `zotero_add_by_url` | Add paper by URL (arXiv, DOI links, web pages) |
| `zotero_add_from_file` | Add item from local PDF/EPUB |
| `zotero_create_note` | Create notes attached to items |
| `zotero_create_annotation` | Highlight text in PDFs |
| `zotero_switch_library` | Switch active library context |

## Example Workflows

### Add metadata to a standalone PDF attachment

When a PDF is imported as a standalone attachment (no parent item), it can't hold
metadata fields like date, publisher, or ISSN. You need to create a proper parent item:

```
1. Use zotero_add_from_file to create a new item from the PDF
2. Use zotero_update_item to fill in metadata
3. Use zotero_manage_collections to place it in the right collection
4. Remove the old standalone attachment from the collection
```

### Organize items into collections

```
1. Use zotero_get_collections to find collection keys
2. Use zotero_manage_collections with add_to/remove_from arrays
```

### Find and merge duplicates

```
1. Use zotero_find_duplicates to list duplicate groups
2. Use zotero_merge_duplicates with confirm=false for a dry-run preview
3. Use zotero_merge_duplicates with confirm=true to execute
```

## Zotero Item Types

Zotero has a fixed set of item types — **custom types cannot be created**. For magazines, the
closest type is `magazineArticle`, which supports fields like publication title, publisher,
ISSN, and date. Use tags to further categorize items (e.g., `magazine-issue` for full issues
vs individual articles).

Common item types: `journalArticle`, `book`, `bookSection`, `magazineArticle`,
`newspaperArticle`, `thesis`, `report`, `document`, `webpage`, `presentation`,
`videoRecording`, `audioRecording`, `patent`, `computerProgram`.

## Zotero Local API

Zotero exposes a read-only REST API at `http://localhost:23119` when running:

- `GET /api/users/0/items` — list items
- `GET /api/users/0/collections` — list collections
- `GET /api/users/0/items/<key>` — get item details

The local API does **not** support POST, PATCH, or DELETE. Write operations go through
the Zotero Web API (`https://api.zotero.org/`), which is what the MCP server uses in
hybrid mode.

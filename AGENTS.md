# Zotero MCP Agent Guide

Instructions and workarounds for Claude Code when working with the Zotero MCP server.

## Known Limitations & Workarounds

### 1. Standalone attachments can't hold metadata

**Problem:** PDFs imported directly into Zotero become `attachment` item types. These only
support `title` and `tags` — fields like `date`, `publicationTitle`, `publisher`, `ISSN`,
and `language` are silently skipped when updating.

**Workaround:** Use `zotero_add_from_file` to create a new parent item from the PDF, then
update the parent's metadata with `zotero_update_item`. Remove the old standalone
attachment from its collection with `zotero_manage_collections`.

**Important:** `zotero_add_from_file` creates a new attachment entry in Zotero storage.
If the source file is already in Zotero's storage (e.g., `/path/to/zotero/storage/<KEY>/file.pdf`),
the new entry gets its own storage key but may not copy the file. Verify the file exists
at the new path (`/path/to/zotero/storage/<NEW_KEY>/file.pdf`) and copy it manually if
missing:

```bash
mkdir -p ~/path/to/zotero/storage/<NEW_KEY>
cp ~/path/to/zotero/storage/<OLD_KEY>/file.pdf ~/path/to/zotero/storage/<NEW_KEY>/file.pdf
```

### 2. No "create item" tool

**Problem:** The MCP server has no generic "create item" tool. You cannot create an empty
`magazineArticle`, `book`, or other item type from scratch.

**Workaround:** Use one of the available "add" tools:

- `zotero_add_from_file` — create an item from a local PDF/EPUB (set `item_type` param)
- `zotero_add_by_doi` — create from a DOI
- `zotero_add_by_url` — create from a URL

Then update metadata with `zotero_update_item`.

### 3. No reparent attachment tool

**Problem:** There is no MCP tool to change an attachment's parent item.

**Workaround:** Instead of reparenting, create a new parent item using `zotero_add_from_file`
which automatically creates and attaches a new copy of the PDF. Then remove the old
standalone attachment from collections.

### 4. No delete item tool

**Problem:** The MCP server cannot delete individual items.

**Workaround:** Use `zotero_merge_duplicates` to trash items — pass the item to keep as
`keeper_key` and the item(s) to remove as `duplicate_keys`. Trashed items can be restored
from Zotero's Trash. For standalone cleanup, remove items from collections with
`zotero_manage_collections` and let the user delete manually in Zotero.

### 5. Custom item types don't exist

**Problem:** Zotero's item types are hardcoded. You cannot create types like "Magazine" or
"Podcast".

**Workaround:** Use the closest built-in type and differentiate with tags:

| Use case | Item type | Suggested tags |
|---|---|---|
| Magazine issue | `magazineArticle` | `magazine-issue` |
| Podcast episode | `audioRecording` | `podcast` |
| Blog post | `blogPost` | — |
| YouTube video | `videoRecording` | `youtube` |

### 6. Write operations fail in local-only mode

**Problem:** Any write tool returns "Cannot perform write operations in local-only mode."

**Workaround:** Both `ZOTERO_API_KEY` and `ZOTERO_LIBRARY_ID` must be set in the MCP
server's `env` config. The library ID is the numeric user ID (not the username), found
at https://www.zotero.org/settings/keys.

### 7. Symlink conflicts after upgrading

**Problem:** If `zotero-mcp-server` was previously installed with `uv` and later with
`pipx`, the symlink at `~/.local/bin/zotero-mcp` may still point to the old `uv` install.
Running `zotero-mcp version` shows an outdated version even though pipx reports the latest.

**Workaround:**

```bash
ln -sf ~/.local/pipx/venvs/zotero-mcp-server/bin/zotero-mcp ~/.local/bin/zotero-mcp
```

### 8. Changes via Web API don't appear locally immediately

**Problem:** Write operations go through the Zotero Web API. The local Zotero instance
won't reflect these changes until it syncs.

**Workaround:** Hit the **Sync** button in Zotero (green arrow, top right) after making
changes through Claude Code.

## Conventions

- Always use the **MCP tools** for Zotero operations. Avoid falling back to raw `curl`
  calls to the local or web API.
- When creating items for full magazine issues, use `magazineArticle` as the item type
  with tags `magazine` and `technology` (or relevant subject tags).
- Place magazine items in the appropriate subcollection (e.g., `Wired` under `Magazines`),
  not the parent collection.
- After using `zotero_add_from_file`, always verify the PDF file exists at the new storage
  path. Copy it from the old location if missing.

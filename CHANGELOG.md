# Changelog

All notable changes to RoslynMCP will be documented in this file.

### v1.17.6 — JSON-RPC Case-Insensitive Parsing
- **Fixed:** JSON-RPC deserialization now accepts both `camelCase` and `PascalCase` property names (e.g., both `"jsonrpc"` and `"JsonRpc"`). Improves compatibility with various MCP clients.

### v1.17.5 — Documentation & Onboarding
- **Improved:** Added "TL;DR: How to Connect" section with quick setup guide and AI teaching instructions.
- **Fixed:** Global memories (`scope:"global"`) no longer get `sessionId` attached — truly session-independent now.

### v1.17.4 — Graph Build Counters Fix
- **Fixed:** `graph_build_from_type` now correctly counts nodes and edges. `nodesSkipped` renamed to `edgesSkipped`.

### v1.17.3 — Custom Tool Descriptions Fix
- **Fixed:** `search_tools` now uses custom descriptions from `config_tool_enabled`.

### v1.17.2 — Knowledge Base (Persistent Long-Term Memory)
- **New:** `kb_*` tools for storing knowledge that persists across sessions forever.
- **New types:** Note, Idea, Doc, Snippet, Link, History, Todo, Reference
- **New tools:** `kb_add`, `kb_get`, `kb_update`, `kb_delete`, `kb_search`, `kb_list`, `kb_tree`, `kb_related`

### v1.17.1 — Tool Schema & Vector Sync
- **Fixed:** AlwaysLoadedTools now show full JSON schema (was minimal `{type:object}`)
- **Optimized:** Removed `validate_text`, `reload_file`, `get_errors` from always-loaded
- **New:** `RegenerateToolVector` — vectors update immediately when description changes

### v1.17.0 — Teaching Claude
- **Fixed:** `memory_forget_session(deleteSession=true)` now correctly deletes ALL session memories
- **New:** Added guidance for teaching Claude how to use RoslynMCP effectively via memory
- **Architecture:** Only 4.6k tokens for 86 tools via `search_tools` → `call_tool` pattern

### v1.16.2
- **New:** `config_tool_enabled` — Enable/disable tools to save memory
- **New:** Custom tool descriptions stored in DB

### v1.16.1
- **Improved:** `get_dependency_graph` — Full project graph with edges, new params: `rootProject`, `maxDepth`, `direction`
- **Fixed:** `memory_*` tools now accept string importance ("high"→8, "medium"→5, "low"→3)

### v1.16.0 — Memory & Graph
- **New:** `memory_*` tools (14 tools) — Persistent memory across sessions
- **New:** `graph_*` tools (10 tools) — Track dependencies, changes, cause-effect relationships
- **New:** `graph_build_from_type` — Auto-build dependency graph from Roslyn analysis
- **Database:** `.roslyn-mcp/memory.db` (SQLite) in solution directory
- **Vector Search:** Uses ONNX model (`all-MiniLM-L6-v2`) for semantic memory search

### v1.14.1
- **Optimized:** Reduced AI context usage (from ~40K to ~33K tokens)
- **Removed:** `change_scope`, `find_circular_dependencies`, `list_refactoring_services`

### v1.13.2
- **New:** `extract_interface` - Extract interface from class for DI/testability
- **New:** `organize_usings` - Sort and remove unused using directives

### v1.13.0
- **License:** Changed from MIT to PolyForm Noncommercial 1.0.0
  - Free for personal, educational, and non-commercial use
  - Commercial use requires separate license
- **New:** `preview_split_class` / `apply_split_class` - Split large classes into partial files with smart method grouping
- **New:** `preview_move_type` / `apply_move_type` - Move types to their own files
- **New:** `apply_extract_method` - Extract code selection into a new method
- **Improved:** `get_type_members` - Now works with ANY type including external (NuGet, System.*, Microsoft.*)
- **Improved:** Tool descriptions with concrete examples for better AI tool selection
- **Removed:** `get_completions`, `get_signature_help` - Replaced by enhanced `get_type_members` and `get_overloads`

### v1.12.0
- **New:** `get_full_context` - Get complete context for a symbol in one call: callers, callees, dependencies, related types
- **Improved:** Redesigned all tool descriptions for better AI tool selection

### v1.11.0
- **New:** `get_completions` - Get code completion suggestions at any position (like IntelliSense)
- **New:** `get_signature_help` - Get method signature and parameter info during calls
- **New:** `apply_rename` - Rename symbols across entire solution with Roslyn accuracy

### v1.10.0
- **Fixed:** `reload_file` now works correctly (was failing with "TryApplyChanges cannot be called from a background thread")

### Initial Release
- Basic MCP server functionality
- Roslyn workspace integration
- HTTP server for AI assistant connection

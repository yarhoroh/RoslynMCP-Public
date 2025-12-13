# Changelog

All notable changes to RoslynMCP will be documented in this file.

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

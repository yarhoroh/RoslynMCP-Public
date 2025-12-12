# Changelog

All notable changes to RoslynMCP will be documented in this file.

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

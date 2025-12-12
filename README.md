# RoslynMCP

MCP server that connects AI assistants to Visual Studio's Roslyn compiler for **100% accurate C# code analysis**.

> **Platform:** Windows only | **Language:** C# only

---

☕ **Like it?** Buy me a coffee so I can mass-produce more extensions at 3 AM → [ko-fi.com/yaroslavhorokhov](https://ko-fi.com/yaroslavhorokhov)

---

## Requirements

- Windows 10/11
- Visual Studio 2022 (17.9 or later)
- C# solution (.sln)

## Supported AI Assistants

- Claude Code (Anthropic)
- GitHub Copilot (with MCP support)
- Cursor
- Any MCP-compatible client

## Why RoslynMCP?

AI assistants typically use text-based search (grep) which produces false positives and misses semantic relationships. RoslynMCP uses the same Roslyn compiler that powers Visual Studio's IntelliSense.

| Criteria | RoslynMCP | Grep/Glob |
|----------|-----------|-----------|
| Symbol search accuracy | ✅ 100% (semantic) | ⚠️ ~80% (text-based) |
| Distinguishes overloads | ✅ Yes | ❌ No |
| Understands types | ✅ Yes | ❌ No |
| Call hierarchy | ✅ Callers/Callees | ❌ No |
| Interface implementations | ✅ Yes | ⚠️ Regex hacks |
| Method overrides | ✅ Yes | ⚠️ Regex hacks |
| False positives | ✅ 0% | ⚠️ Possible (comments, strings) |

## When to Use

**Use RoslynMCP for:**
- Find all usages of a symbol (class, method, property)
- Refactoring - find all places that need to change
- Code understanding - type hierarchy, call graphs
- Analyze method overloads
- Find interface implementations
- Impact analysis - what breaks if I change this?

**Use Grep/Glob for:**
- Quick text search - strings, comments, TODO
- Non-C# files - XML, JSON, config
- Regex patterns
- Multi-file-type search

## Installation

1. Install RoslynMCP from [Visual Studio Marketplace](https://marketplace.visualstudio.com/)
2. Open your C# solution in Visual Studio
3. Go to **View → Other Windows → RoslynMCP**
4. Copy the configuration for your AI assistant

## Features

RoslynMCP provides **46 Roslyn-powered tools** including:

### v1.10.0
- **Fixed:** `reload_file` now works correctly (was failing with "TryApplyChanges cannot be called from a background thread")

- **Navigation:** Find references, definitions, implementations, callers/callees
- **Analysis:** Type info, class hierarchy, code metrics, data flow
- **Diagnostics:** Real-time errors, warnings, async issues, performance problems
- **Refactoring:** Rename preview, impact analysis, extract method feasibility

## Troubleshooting

### Extension not loading
- Check Visual Studio Output window for errors
- Verify extension is enabled in Extensions → Manage Extensions

### AI assistant not connecting
- Ensure Visual Studio has a solution open
- Open the RoslynMCP tool window (View → Other Windows → RoslynMCP)
- Verify the server status shows "Running"

### No tools available
- Make sure you copied the correct configuration for your AI assistant
- Restart your AI assistant after configuration changes

## License

MIT

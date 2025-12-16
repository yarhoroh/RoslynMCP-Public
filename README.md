<p>
  MCP server that connects AI assistants to Visual Studio's Roslyn compiler for <strong>100% accurate C# code analysis</strong>.
</p>

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

1. Install RoslynMCP from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=YaroslavHorokhov.RoslynMcp/)
2. Open your C# solution in Visual Studio
3. Go to **View → Other Windows → RoslynMCP**
4. Copy the configuration for your AI assistant

## Features

### v1.14.0
- **Optimized:** Reduced AI context usage (from ~40K to ~33K tokens) by streamlining tool descriptions
- **Removed:** `change_scope` (use `impact_analysis`), `find_circular_dependencies`, `list_refactoring_services` - rarely used tools

### v1.13.2
- **New:** `extract_interface` - Extract interface from class for DI/testability. Generates `IClassName.cs` and adds interface implementation to class
- **New:** `organize_usings` - Sort and remove unused using directives. Roslyn-powered cleanup with CS8019/CS0105 diagnostics

### v1.13.0
- **New:** `preview_split_class` / `apply_split_class` - Split large classes into partial files with smart method grouping
- **New:** `preview_move_type` / `apply_move_type` - Move types to their own files
- **New:** `apply_extract_method` - Extract code selection into a new method
- **Improved:** `get_type_members` - Now works with ANY type including external (NuGet, System.*, Microsoft.*). Use short name (`Solution`) or full name (`Microsoft.CodeAnalysis.Solution`)
- **Improved:** Tool descriptions with concrete examples for better AI tool selection
- **Removed:** `get_completions`, `get_signature_help` - Replaced by enhanced `get_type_members` and `get_overloads`

### v1.12.0
- **New:** `get_full_context` - Get complete context for a symbol in one call: callers, callees, dependencies, related types
- **Improved:** Redesigned all tool descriptions for better AI tool selection - priorities, explicit negations, use cases, workflow guidance

### v1.11.0
- **New:** `get_completions` - Get code completion suggestions at any position (like IntelliSense)
- **New:** `get_signature_help` - Get method signature and parameter info during calls
- **New:** `apply_rename` - Rename symbols across entire solution with Roslyn accuracy

### v1.10.0
- **Fixed:** `reload_file` now works correctly (was failing with "TryApplyChanges cannot be called from a background thread")

RoslynMCP provides **50+ Roslyn-powered tools**:

| Category | Tools |
|----------|-------|
| ***🔍 Navigation*** | `find_references`, `find_definition`, `find_callers`, `find_callees`, `find_implementations`, `find_overrides`, `find_base_members` |
| ***🧠 Understanding*** | `understand_type`, `understand_method`, `get_type_info`, `get_type_members`, `get_method_body`, `get_class_hierarchy` |
| ***📁 Structure*** | `get_solution_structure`, `get_project_structure`, `get_file_outline`, `get_types_in_file`, `find_entry_points`, `get_dependency_graph` |
| ***🩺 Diagnostics*** | `get_errors`, `get_warnings`, `validate_text`, `find_async_issues`, `find_performance_issues`, `find_unused_code` |
| ***🔧 Refactoring*** | `preview_rename`, `apply_rename`, `extract_interface`, `organize_usings`, `impact_analysis`, `preview_extract_method`, `suggest_refactorings` |
| ***💡 IntelliSense*** | `get_quick_fixes`, `get_overloads`, `get_xml_documentation` |
| ***🔎 Search*** | `find_attribute_usages`, `find_event_subscribers`, `find_extension_methods`, `find_tests_for_type`, `text_search` |
| ***📊 Metrics*** | `get_code_metrics`, `analyze_data_flow`, `get_full_context`, `get_constructor_parameters` |

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

**PolyForm Noncommercial 1.0.0** — Free for personal and non-commercial use.

Commercial use requires a separate license. See [LICENSE](https://github.com/yarhoroh/RoslynMCP-Public/blob/main/LICENSE) for details.

For commercial licensing, contact: [ko-fi.com/yaroslavhorokhov](https://ko-fi.com/yaroslavhorokhov)

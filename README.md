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
- Codex
- Any MCP-compatible client

## Why RoslynMCP?

AI assistants use text search (grep/ripgrep) which misses semantic relationships. RoslynMCP provides compiler-level accuracy using the same Roslyn engine that powers Visual Studio IntelliSense.

| Capability | RoslynMCP | Text Search |
|------------|-----------|-------------|
| Symbol resolution | Semantic (100%) | Text matching (name collisions) |
| Overloads | Distinguishes all | Cannot distinguish |
| Type hierarchy | Full inheritance tree | Not available |
| Call graph | Callers & callees | Not available |
| Refactoring | Safe rename across solution | Find & replace (risky) |

**Use RoslynMCP:** symbol usages, type hierarchies, call graphs, safe refactoring, impact analysis, persistent memory

## Installation

1. Install RoslynMCP from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=YaroslavHorokhov.RoslynMcp)
2. Open your C# solution in Visual Studio
3. Go to **View → Other Windows → RoslynMCP**
4. Copy the configuration for your AI assistant

## Features

### v1.17.0 — Teaching Claude
- **Fixed:** `memory_forget_session(deleteSession=true)` now correctly deletes ALL session memories regardless of `keepGlobal` flag
- **New:** Added guidance for teaching Claude how to use RoslynMCP effectively via memory
- **Architecture:** Only 4.6k tokens for 86 tools via `search_tools` → `call_tool` pattern (vs ~40k+ if all tools loaded)

#### How Tool Search Works

All 86 tool descriptions are indexed as 384-dimensional vectors using `all-MiniLM-L6-v2` ONNX model. When you call `search_tools("find usages")`, the query is embedded and matched via cosine similarity.

```
search_tools("find usages") → embedding → cosine similarity → top-K tools
```

**Config options** (via `config_set`):
| Key | Default | Description |
|-----|---------|-------------|
| `tool_search_threshold` | 0.3 | Minimum similarity score (0-1) |

#### Two Graph Systems

RoslynMCP uses two complementary graph systems:

| Graph | Tools | Purpose |
|-------|-------|---------|
| **Roslyn Call Graph** | `find_callers`, `find_callees`, `understand_method` | Static code analysis — who calls what |
| **DevGraph** | `graph_*` tools | Dynamic tracking — changes, causes, decisions |

**Roslyn Graph** is built automatically from code. Use it to navigate:
```
find_callers("SaveUser") → which methods call SaveUser?
find_callees("SaveUser") → what does SaveUser call?
understand_method("SaveUser") → callers + callees + body in one call
```

**DevGraph** is built during work. Use it to track:
```
graph_track_change("User.cs", "Added validation")
graph_track_cause(errorNodeId, "Fixed null reference")
graph_get_impact("UserService") → what breaks if I change this?
```

#### Teaching Claude via Memory

Write rules to memory so Claude remembers how to work with your codebase. Use `memory_learn` for global rules:

```
memory_learn("For C# code: ONLY use Roslyn MCP tools, NEVER grep/Glob. validate_text BEFORE edit, reload_file AFTER.")
memory_learn("Write memories in English (better vector search). Show user info in their language.")
memory_learn("Use graph tools with sessions: start_session before tracking, graph_track_change after edits, graph_track_cause for bug fixes.")
```

**Recommended starter memories:**
| Rule | Why |
|------|-----|
| Use Roslyn tools for C# | 100% accurate vs text search |
| validate_text before edit | Catch errors before saving |
| reload_file after edit | Sync Roslyn workspace |
| English in memory | Vector search works better |
| Graph with sessions | Track changes, dependencies, causes |

After `memory_learn`, Claude will recall these rules via `memory_context` at conversation start.

### v1.16.2
- **New:** `config_tool_enabled` — Enable/disable tools to save memory. Disabled tools won't load after server restart
- **New:** Custom tool descriptions — LLM can update tool descriptions stored in DB (used in `tools/list` response)
- **New:** `config_list` now shows all tools with enabled/disabled status and stats

### v1.16.1
- **Improved:** `get_dependency_graph` — Full project graph with edges by default. New params: `rootProject` (filter), `maxDepth`, `direction` (dependencies/dependents/both) for large solutions (70+ projects)
- **Fixed:** `memory_*` tools now accept string importance ("high"→8, "medium"→5, "low"→3) in addition to numbers
- **Fixed:** `memory_update` accepts both `id` and `memoryId` parameters

### v1.16.0 — Memory & Graph (Experimental)
- **New:** `memory_*` tools (14 tools) — Persistent memory across sessions
- **New:** `graph_*` tools (10 tools) — Track dependencies, changes, cause-effect relationships
- **New:** `graph_build_from_type` — Auto-build dependency graph from Roslyn analysis (hierarchy, callers, callees)
- **Database:** `.roslyn-mcp/memory.db` (SQLite) in your solution directory
- **Vector Search:** Uses ONNX model (`all-MiniLM-L6-v2`) for semantic memory search
- **Config:** LLM can tune vector search via `config_set` (e.g., `vector_threshold`, `scoring_similarity_weight`)

> ⚠️ **Experimental:** Memory and Graph tools are new. We're still figuring out the best workflows. Try different approaches and find what works for you!

#### Memory Scopes & Importance
| Scope | Lifetime | Use for |
|-------|----------|---------|
| `global` | Forever | Project rules, patterns, architectural decisions |
| `session` | Until session ends | Current task context, temporary notes |

**Importance (1-10):** Higher = more likely to appear in `memory_context`. Use 8-10 for critical rules, 5-7 for useful info, 1-4 for minor notes.

#### Memory Tools
| Tool | Purpose |
|------|---------|
| `memory_start_session` | Start tracking a dev session |
| `memory_end_session` | End session with summary |
| `memory_list_sessions` | List all sessions |
| `memory_restore` | Restore context from previous session |
| `memory_remember` | Save decision/change/error/fix |
| `memory_learn` | Save pattern/lesson (global) |
| `memory_recall` | Get memories by type/tags/files |
| `memory_context` | Get relevant context for task |
| `memory_search` | Semantic search in memories |
| `memory_update` | Update existing memory |
| `memory_consolidate` | Merge similar memories |
| `memory_analyze` | Memory health stats |
| `memory_forget` | Delete a memory |
| `memory_forget_session` | Delete session + its graph nodes |

#### Graph Tools
| Tool | Purpose |
|------|---------|
| `graph_track_change` | Track file modification |
| `graph_track_dependency` | Track A depends on B |
| `graph_track_cause` | Track cause → effect |
| `graph_build_from_type` | Auto-build graph from Roslyn (hierarchy + calls) |
| `graph_get_impact` | What breaks if I change X? |
| `graph_get_dependencies` | Show dependency tree |
| `graph_get_path` | Find path between two nodes |
| `graph_get_history` | File change history |
| `graph_visualize` | Generate Mermaid diagram |
| `graph_delete_node` | Remove node + orphans |

#### Quick Start
```
# Start session
memory_start_session("Adding feature X")

# Track changes as you work
graph_track_change("MyClass.cs", "Added new method")
graph_track_dependency("ServiceA", "ServiceB")

# Or auto-build from existing code
graph_build_from_type("MyService")  # builds hierarchy + callers + callees

# Save learnings
memory_learn("Always validate input before processing")

# End session
memory_end_session("Feature X completed")

# Later: recall context
memory_context(forTask="similar feature")
graph_get_impact(nodeId="ServiceB")
```

### v1.14.1
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

RoslynMCP provides **70+ Roslyn-powered tools**:

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
| ***🧠 Memory*** | `memory_start_session`, `memory_end_session`, `memory_remember`, `memory_learn`, `memory_recall`, `memory_context`, `memory_search`, `memory_forget`, `memory_forget_session`, `memory_update`, `memory_consolidate`, `memory_analyze`, `memory_list_sessions`, `memory_restore` |
| ***🔗 Graph*** | `graph_track_change`, `graph_track_dependency`, `graph_track_cause`, `graph_build_from_type`, `graph_get_impact`, `graph_get_history`, `graph_get_dependencies`, `graph_get_path`, `graph_visualize`, `graph_delete_node` |

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

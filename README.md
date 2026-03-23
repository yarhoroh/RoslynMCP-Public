#  Give your AI assistant full control over Visual Studio — code analysis, debugging, UI automation, and more.

*Make AI coding assistants actually useful for professional C# development.*

> **Platform:** Windows only | **Language:** C# only | **Requirements:** Windows 10/11, Visual Studio 2022 (17.9+), any MCP-compatible AI assistant

## Demo Video
https://www.youtube.com/watch?v=skvnHbm2lpk

## Quick Start

1. Install from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=YaroslavHorokhov.RoslynMcp)
2. Open your C# solution in Visual Studio
3. Go to **View → Other Windows → RoslynMCP Dashboard**
4. Copy the connection command for your AI assistant (Claude Code, Codex, Copilot, or Cursor)
5. Ask your AI to explore all available tools — warning: it may refuse to work without them ever again 😄

> ⚠️ The path is unique per VS installation — always copy from the Dashboard, not from examples.

**Multiple VS instances:** Each Visual Studio window runs its own MCP server. AI auto-connects to the one matching your working directory.

**Tip:** Create [skills](https://docs.anthropic.com/en/docs/claude-code/skills) (slash commands) for the tools you use most — this dramatically improves tool selection and output quality.

## Configuration Examples

Open the **RoslynMCP Dashboard** in Visual Studio (**View → Other Windows → RoslynMCP Dashboard**) and copy the connection command for your AI tool.

> ⚠️ The path is unique per VS installation — **always copy from the Dashboard**, not from examples below.

**Claude Code / Codex:**
```bash
claude mcp add roslyn -- "C:\Users\YOU\...\Proxy\RoslynMcp.Proxy.exe"
codex mcp add roslyn -- "C:\Users\YOU\...\Proxy\RoslynMcp.Proxy.exe"
```

Works with any AI client that supports the [Model Context Protocol](https://modelcontextprotocol.io/).

## Features

> **RoslynMCP includes a 30-day free trial.** After the trial period, a license is required to continue using the tools.

### Core Features

#### Roslyn Code Analysis
- **130+ tools** loaded on-demand via `search_tools` → `call_tool`
- **Navigation:** `find_references`, `find_definition`, `find_callers`, `find_callees`, `find_implementations`, `find_overrides`
- **Understanding:** `understand_type`, `understand_method`, `get_type_info`, `get_type_members`, `get_method_body`
- **Diagnostics:** `get_errors`, `get_warnings`, `validate_text`, `find_async_issues`, `find_performance_issues`
- **Refactoring:** `apply_rename`, `extract_interface`, `organize_usings`, `apply_split_class`, `apply_extract_method`
- **Structure:** `get_solution_structure`, `get_project_structure`, `get_file_outline`, `get_dependency_graph`

#### OOP C# Programming (`cs` tool)

- **86 actions** for object-oriented C# programming directly through Roslyn API — no text editing, no file I/O
- Create/update/delete types, members, constructors, properties, events, attributes
- Statement-level editing inside method bodies at any nesting depth (if/for/try/switch blocks)
- Expression builders, batch mode, block path navigation (`Type.Method.if[0].else`)
- Every mutation returns instant Roslyn diagnostics — no build needed to catch errors

#### Visual Studio IDE Control

- `vs` — 40 IDE actions: build, debug (start/stop/step/breakpoints), file operations, find/replace, deploy, bookmarks, run_tests, configuration
- `vs_query` — 25 IDE queries: solution, projects, errors, build output, debug state, locals, callstack, threads, expressions, editor context, bookmarks, tests, test_results
- `vs_query editor_context` — cursor position, selected text, word at cursor, surrounding code, breakpoint status, debug value + Roslyn semantic context (method, class, namespace, scope chain, symbol info, diagnostics)
- `bookmark_set` / `bookmark_next` / `bookmark_prev` / `bookmark_clear_all` — AI sets labeled bookmarks, navigates between them, user sees them in VS editor margins
- `debug_monitor` — live state of debugged app: windows, dialogs, UI elements. Blocking `waitFor` eliminates polling
- `list_instances` / `switch_instance` — manage multiple VS instances from one AI session
- **Auto-schema on errors** — when AI sends wrong parameters, the error response includes the correct JSON schema so AI self-corrects on next attempt


#### Test Runner

- `vs { "action": "run_tests" }` — run all tests via Test Explorer (non-blocking, background)
- `vs { "action": "run_tests", "options": {"failedOnly": true} }` — re-run only failed tests
- `vs { "action": "stop_tests" }` — cancel running tests
- `vs_query { "what": "test_results" }` — get results: passed/failed counts, error messages with file:line
- `vs_query { "what": "tests" }` — discover all test methods via Roslyn ([Fact]/[Test]/[TestMethod] attributes)
- `find_tests_for_type` — find unit tests targeting a specific type across test projects
- Full TDD cycle: run → check errors → fix → re-run failed → all green

#### Desktop Automation & UI Testing

- `screen` (23 actions) — screenshots (full screen, window, region), mouse (click/double-click/move/scroll), keyboard input (Unicode, hotkeys, combos), window/process management, clipboard, screen info
- `ui_find` / `ui_invoke` / `ui_tree` — Windows UI Automation: find elements by name/automationId, invoke buttons, toggle checkboxes, select items, double-click
- `ui_set_value` / `ui_get_value` / `ui_table` — read/write form fields, read data grids
- `ui_expand` / `ui_select` — expand/collapse tree nodes, select items in lists and combo boxes
- Works with WinForms, WPF, Win32 apps. Partial support for Electron apps (Teams, Slack, WhatsApp).

#### Memory & Knowledge Base
- `memory_*` (17 tools) — persistent cross-session memory with vector search (ONNX)
- `kb_*` (8 tools) — permanent knowledge base with semantic + full-text search

#### Dashboard

- Tool window in VS (**View → Other Windows → RoslynMCP Dashboard**)
- License status with activation (trial countdown, license key input, Buy button)
- Connection configs for Claude Code, Codex, GitHub Copilot, Cursor — copy with one click
- Tabs: Workflows editor, Memory editor, Knowledge Base editor, Debug Monitor (live windows/elements/events)

### Experimental Features

> These features are functional but still being improved and tested.

#### Claude Chat Panel

- Built-in Claude Code chat directly in Visual Studio (**View → Other Windows → Claude Chat**)
- Adapted from [Claude Code for VS Code](https://marketplace.visualstudio.com/items?itemName=anthropic.claude-code) extension — React UI running in WebView2
- RoslynMCP tools are automatically connected via `--mcp-config` — no manual setup needed
- Bundled Roslyn skills are included with the extension at `Skills/.claude/skills/` inside the VSIX. To customize, copy them to your project's `.claude/skills/` directory or edit them for use with Claude Code in PowerShell


#### Markdown Tool
- `md` — structured .md editing (9 actions): `toc`, `read`, `read_section`, `search`, `edit_section`, `insert_section`, `append`, `delete_section`, `replace`

#### DevGraph
- `graph_*` (10 tools) — track changes, dependencies, cause-effect during development
- `graph_build_from_type` — auto-build dependency graph from Roslyn semantic model
- `graph_track_change` / `graph_track_dependency` / `graph_track_cause` — record changes and relationships
- `graph_get_impact` — what breaks if I change X?
- `graph_get_dependencies` / `graph_get_path` / `graph_get_history` — explore connections and evolution
- `graph_visualize` — export as Mermaid or DOT diagram

#### AI Workflow Engine
- `wf_*` (27 tools) — reusable step-by-step instructions for AI
- AI follows steps, annotates results, learns from mistakes
- Step types: execute, verify, wait, call_case (nested), ai_freeform
- Background watchers for windows, UI elements, processes
- Progress tracking and run history

## Troubleshooting

### Extension not loading
- Check Visual Studio Output window for errors
- Verify extension is enabled in Extensions → Manage Extensions

### AI assistant not connecting

- Ensure Visual Studio has a solution open
- Open the RoslynMCP Dashboard (View → Other Windows → RoslynMCP Dashboard) — status should show "Running"
- Copy the connection command from the Dashboard for your AI tool

**First connection fails (e.g. in VS Copilot logs):**
This is normal — the MCP server starts before the extension finishes loading. The automatic retry connects successfully. No action needed.

**Multiple VS instances or wrong project:**
Ask your AI assistant: *"List all active RoslynMCP instances and switch to my current project."*
The assistant will use `list_instances` and `switch_instance` to reconnect to the right Visual Studio instance.

### No tools available
- Make sure you copied the correct configuration for your AI assistant
- Restart your AI assistant after configuration changes


### Data Storage

RoslynMCP creates databases in `.roslyn-mcp/` inside your solution directory:

| File | Contents |
|------|----------|
| `memory.db` | Memory, Knowledge Base, DevGraph, configuration |
| `testcases.db` | AI Workflow Engine: workflows, steps, run history |

License data is stored in `%AppData%/RoslynMcp/`.


## RoslynMCP vs Other AI IDE Integrations


All major AI coding tools (Claude Code, GitHub Copilot, Cursor) provide code understanding, inline diffs, terminal access, and agent capabilities. RoslynMCP is different in **two specific ways**:

### Direct Roslyn Compiler Access

Other tools analyze code through **text search or LLM context windows**. RoslynMCP connects to the **same Roslyn compiler API** that powers Visual Studio IntelliSense:

| Capability | Text/LLM-based tools | **RoslynMCP** |
|---|---|---|
| Find references | Text search — may miss overloads, generics, partial classes | `find_references` — compiler-resolved, 100% accurate |
| Understand type | LLM reads source text | `understand_type` — members, hierarchy, callers from compiled model |
| Call graph | Not available | `get_full_context` — recursive call tree up & down |
| Refactoring | Generate text diff | `apply_rename`, `apply_extract_method`, `apply_split_class` — via Roslyn code actions |
| Edit C# code | Generate text diff, hope it compiles | `cs` tool — 86 OOP actions with instant Roslyn diagnostics |
| Symbol at cursor | IDE-internal or unavailable | `editor_context` — resolved type, kind, full name from semantic model |
| Data flow | Not available | `analyze_data_flow`, `analyze_operations` |

### Full Visual Studio Debugger Control

No other AI tool gives the AI model **direct programmatic access** to the Visual Studio debugger:

- `vs start_debug` / `stop_debug` — launch and stop debugging sessions
- `vs step_over` / `step_into` / `step_out` — sync stepping, returns new state + locals automatically
- `vs breakpoint_add` — conditional breakpoints, function breakpoints
- `vs_query locals` / `callstack` / `expression` — inspect runtime state
- `debug_monitor` — live window/dialog tracking with blocking `waitFor`

This enables the **self-development loop**: code → build → set breakpoint → debug → inspect → fix — all controlled by AI.


## License

Starting from v1.18.6, RoslynMCP includes a **30-day free trial**. After the trial, a subscription is required.

- **Trial:** 30 days, full access, no registration needed
- **Subscription:** available at [roslynmcp.lemonsqueezy.com](https://roslynmcp.lemonsqueezy.com)
- **License key:** enter in RoslynMCP Dashboard (View → Other Windows → RoslynMCP Dashboard)
- **Previous versions** (v1.18.4 and earlier) remain fully free

For enterprise licensing or questions, contact via [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=YaroslavHorokhov.RoslynMcp).

## Tired of typing prompts manually? 🎤
Try Murmur — offline voice-to-text for Windows. Fast, private, no cloud.
👉 https://murmurvt.com/

---

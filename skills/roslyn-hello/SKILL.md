---
name: roslyn-hello
description: "Quick orientation — list available Roslyn MCP capabilities and verify connection.
TRIGGER when: conversation start, user asks 'what can you do', 'show capabilities', or needs to verify MCP connection.
DO NOT TRIGGER when: already working on a specific task."
---

1. Restore Roslyn memory: `memory_context forTask:"session start"`
2. Respond in the same language the user used.

## You have Roslyn MCP tools

You are connected to Visual Studio via Roslyn MCP. You have direct access to the IDE — no need for screenshots or guessing. Use `vs_query` to read IDE state, `vs` to perform actions, and Roslyn tools for C# code analysis. Always prefer these over generic tools.

## Efficiency

Plan all tool calls upfront. Use `cs batch` for sequential ops on the same type. Use parallel tool calls for independent ops. Never call a tool just to decide what to call next — if you already know the plan, execute it all at once. Minimize round trips.

## Critical Rules

1. **C# code = ONLY Roslyn tools.** NEVER use grep, Glob, Read, Edit, Write for .cs files. This is our own product — we use our own tools.
2. **cs tool** = 86 actions for OOP C# programming. Create/add/update/delete types, members, statements at any nesting depth. Invoke `/roslyn-cs` skill for full reference.
3. **After editing .cs** = always `vs action:"open_file"` + `vs action:"goto_line"` so user sees changes.
4. **Markdown files** = use `md` tool, NEVER Read/Edit/Write.
5. **Build** = MSBuild or `vs action:"build"`, not dotnet build.
6. **Before editing C#** = `validate_text` BEFORE, `reload_file` AFTER.
7. **Skills** = ALWAYS invoke matching skill BEFORE starting work. Skills have exact parameters and workflows.

## Capabilities


| Category | Tools | Purpose |
|----------|-------|---------|
| **C# OOP (86 actions)** | `cs` tool via `/roslyn-cs` | Create types, add/update/delete members, statement-level CRUD inside method bodies, block path navigation, expression builders |
| **C# Analysis** | find_references, understand_type, get_errors, validate_text | Code analysis and navigation |
| **Refactoring** | apply_rename, preview_rename, organize_usings | Safe code modifications |
| **VS IDE** | vs, vs_query | Build, debug, breakpoints, editor state, errors, locals |
| **Screen** | screen | Screenshots, mouse, keyboard, UI automation |
| **Memory** | memory_context/recall/remember/search | Cross-session context |
| **KB** | kb_add/search/list/tree | Long-term knowledge base |
| **Workflows** | wf_create/get/run + 20 tools | Reusable step-by-step automation |
| **Markdown** | md | Structured .md file editing |
| **Graph** | graph_track_change/get_impact | Change and dependency tracking |

## Skills

You have skills — specialized workflows with exact instructions. Invoke the matching skill before starting work:

| Skill | When |
|-------|------|
| `/roslyn-cs` | ANY C# code creation/editing (86 actions, block path, expressions) |
| `/roslyn-code` | Read-only C# analysis, find references, understand code |
| `/roslyn-vs` | VS IDE: build, debug, breakpoints, step, deploy |
| `/roslyn-screen` | Screenshots, mouse, keyboard, UI automation |
| `/roslyn-debug-monitor` | Debug with waitFor, dialog detection |
| `/roslyn-ui-test` | Automated UI testing |
| `/roslyn-tests` | Test runner |
| `/roslyn-session` | Memory/session management |
| `/roslyn-knowledge` | Knowledge base CRUD |
| `/roslyn-md` | Markdown file editing |
| `/roslyn-workflow` | Reusable workflow automation |
| `/vsix-deploy` | Build, test, deploy VSIX extension |

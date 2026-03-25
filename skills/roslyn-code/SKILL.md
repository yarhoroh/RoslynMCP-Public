---
name: roslyn-code
description: "Full C# code workflow with Roslyn MCP — orientation, understanding, references, analysis, edit, verification.\nTRIGGER when: ANY operation on C# code — read, analyze, navigate, refactor, edit, find references, check errors.\nDO NOT TRIGGER when: working with non-C# files, or only VS IDE commands (build/debug)."
---

# Roslyn MCP: Code Workflow

> **Language rule:** Write all memory and KB entries in English. Respond to the user in whatever language they used.

## Workflow

```
Orient → Understand → Find relations → Analyze → Pre-validate → Edit → Reload → Verify
```

---

## 1. Orient

```json
get_solution_structure {}
get_project_structure { "projectName": "<projectName>" }
get_file_outline { "filePath": "<filePath>" }
get_types_in_file { "filePath": "<filePath>" }
get_workspace_status {}
config_list {}
```

---

## 2. Understand

```json
// High-level type analysis (summary with members, inheritance, usage)
understand_type { "typeName": "<typeName>" }

// High-level method analysis (signature, body, callers, callees)
understand_method { "methodName": "<methodName>", "containingType": "<typeName>" }

// Get method source code only
get_method_body { "methodName": "<methodName>", "containingType": "<typeName>" }

// Type members and base types (solution types only)
get_type_info { "typeName": "<typeName>", "includeMembers": true }

// Members of ANY type including NuGet
get_type_members { "typeName": "<typeName>", "includeInherited": true }

// Full inheritance tree (base UP, derived DOWN)
get_class_hierarchy { "typeName": "<typeName>" }

// Constructor parameters (DI analysis)
get_constructor_parameters { "typeName": "<typeName>" }

// All method overloads (solution + NuGet)
get_overloads { "methodName": "<methodName>", "containingType": "<typeName>" }

// Access modifiers (public, private, internal, etc.)
get_accessibility { "symbolName": "<symbolName>", "symbolKind": "any" }

// XML documentation comments (/// summary)
get_xml_documentation { "symbolName": "<symbolName>", "symbolKind": "any" }

// Deep recursive context (callers, callees, deps — LARGE response)
get_full_context { "symbolName": "<symbolName>", "symbolKind": "any", "depth": 2, "maxNodes": 50 }
```

### Overload behavior (v1.18.8+)

`understand_method` and `get_method_body` return **all overloads** when multiple exist:
- Single method → full result (signature, body, callers, callees)
- Multiple overloads → `overloadCount` + `overloads[]` array with signature, parameters, file, line

All tools work correctly with **partial classes** — members found across all partial declarations.

### understand_type vs get_type_info vs get_type_members

| Tool | When to use |
|------|-------------|
| `understand_type` | First look — high-level summary |
| `get_type_info` | Detailed members + base types (solution only) |
| `get_type_members` | Members of ANY type including NuGet/BCL |

---

## 3. Find relations

> **CALL TREE?** → Use `get_full_context` (one call, both directions).
> **Do NOT** loop `find_references` manually to build call trees.

```json
// All usages of a symbol (reads, writes, assignments)
find_references {
  "symbolName": "<symbolName>",
  "symbolKind": "any",
  "includeContext": true,
  "groupBy": "typeAndMember",
  "page": 0, "pageSize": 20
}

// Go to definition
find_definition { "symbolName": "<symbolName>", "symbolKind": "any" }

// Interface implementations / inheritors
find_implementations { "typeName": "<typeName>", "includeIndirect": true }

// Types deriving from base type
find_derived_types { "typeName": "<typeName>", "includeIndirect": true }

// Who calls a method (call graph UP)
find_callers { "methodName": "<methodName>", "containingType": "<typeName>", "includeContext": true }

// What a method calls (call graph DOWN)
find_callees { "methodName": "<methodName>", "containingType": "<typeName>", "includeContext": true }

// Overrides of virtual/abstract method
find_overrides { "methodName": "<methodName>", "containingType": "<typeName>" }

// What a method overrides or implements (go UP the hierarchy)
find_base_members { "memberName": "<memberName>", "containingType": "<typeName>" }

// Event subscribers (+= and -=)
find_event_subscribers { "eventName": "<eventName>", "containingType": "<typeName>" }

// Extension methods for a type
find_extension_methods { "targetType": "<targetType>" }

// Attribute usages ([Obsolete], [Serializable], etc.)
find_attribute_usages { "attributeName": "<attributeName>" }

// Entry points (Main, controllers, HTTP endpoints)
find_entry_points {}

// Project dependency graph
get_dependency_graph {
  "rootProject": "<projectName>",
  "maxDepth": 2,
  "direction": "both",
  "includeDocCount": true
}
```

### Building a call tree

**Preferred:** `get_full_context` — full hierarchy (up AND down) in one call:

```json
get_full_context { "symbolName": "<symbolName>", "symbolKind": "method", "depth": 4, "maxNodes": 100 }
```

**Fallback:** `find_callers` / `find_callees` — one direction only, or when `get_full_context` returns too much noise.

---

## 4. Analyze

```json
// Code metrics (complexity, method length)
get_code_metrics { "symbolName": "<symbolName>", "symbolKind": "method" }

// Deep semantic analysis — all operations step-by-step
analyze_operations {
  "methodName": "<methodName>",
  "containingType": "<typeName>",
  "maxDepth": 10,
  "filterByKind": "Invocation"
}
// filterByKind values: Invocation, Assignment, Binary, Loop, Conversion, ObjectCreation

// Variable data flow in code region
analyze_data_flow { "filePath": "<filePath>", "startLine": 10, "endLine": 50 }

// Impact analysis — what breaks if symbol is changed
impact_analysis { "symbolName": "<symbolName>", "symbolKind": "any" }

// Async/await anti-patterns
find_async_issues { "filePath": "<filePath>" }

// Performance anti-patterns
find_performance_issues { "filePath": "<filePath>" }

// Unused code (private methods, fields)
find_unused_code { "filePath": "<filePath>" }
```

---

## 5. Pre-validate (REQUIRED before saving)

```json
// Validate code BEFORE writing to file
validate_text {
  "filePath": "<filePath>",
  "text": "<codeToValidate>"
}
```

If errors — **do not save**. Fix first.

---

## 6. Edit

Use `Edit` (not `Write`) for changes. After each change:

```json
reload_file { "filePath": "<filePath>" }
```

---

## 7. Verify

```json
// Compilation errors (real-time)
get_errors { "filePath": "<filePath>", "includeWarnings": false }

// Warnings only
get_warnings { "filePath": "<filePath>" }

// Quick fixes for an error
get_quick_fixes { "filePath": "<filePath>", "line": 42, "diagnosticId": "CS0103" }

// Refactoring suggestions at position
suggest_refactorings { "filePath": "<filePath>", "line": 42 }
```

---

## Refactoring

> Always **preview** before **apply**.

```json
// Rename
preview_rename { "symbolName": "<symbolName>", "newName": "<newName>", "symbolKind": "any" }
apply_rename { "symbolName": "<symbolName>", "newName": "<newName>", "symbolKind": "any" }

// Extract method
preview_extract_method { "filePath": "<filePath>", "startLine": 10, "endLine": 20 }
apply_extract_method { "filePath": "<filePath>", "startLine": 10, "endLine": 20, "methodName": "<methodName>" }

// Move type to own file
preview_move_type { "filePath": "<filePath>", "line": 15 }
apply_move_type { "filePath": "<filePath>", "line": 15, "newFileName": "<newFileName>" }

// Split class into partial files
preview_split_class { "filePath": "<filePath>", "className": "<className>", "groupingStrategy": "smart" }
apply_split_class {
  "filePath": "<filePath>",
  "className": "<className>",
  "groups": { "Events": ["OnClick", "OnLoad"], "Data": ["GetData", "SaveData"] },
  "dryRun": false
}

// Extract interface
extract_interface { "typeName": "<typeName>", "interfaceName": "<interfaceName>", "memberNames": ["Method1", "Method2"] }

// Organize usings (sort + remove unused)
organize_usings { "filePath": "<filePath>", "removeUnused": true }
```

---

## Error diagnostics (Apollo)

```json
// Compile with structured diagnostics + fix suggestions
compile_and_diagnose { "filePath": "<filePath>", "includeContext": true, "maxErrors": 20 }

// Auto-repair loop
repair_loop {
  "filePath": "<filePath>",
  "maxIterations": 5,
  "strategy": "sequential",
  "runTests": false,
  "testProjectHint": "<testProject>"
}

// Validate a fix: check regressions, optionally run affected tests
validate_fix {
  "filePath": "<filePath>",
  "newContent": "<fixedCode>",
  "originalDiagnosticIds": ["CS0103"],
  "runAffectedTests": false
}
```

---

## Dependency graph (visual)

```json
graph_build_from_type { "typeName": "<typeName>", "includeCallers": true, "includeCallees": true }
graph_visualize { "format": "mermaid", "depth": 2 }
graph_get_dependencies { "symbolName": "<symbolName>", "depth": 2, "direction": "both" }
graph_get_path { "fromNodeId": "<nodeA>", "toNodeId": "<nodeB>", "maxDepth": 5 }
```

---

## Text search (fallback — for comments, strings, SQL, JSON)

```json
text_search {
  "query": "<searchText>",
  "queryMode": "literal",
  "filePattern": "*.cs",
  "wholeWord": false,
  "caseSensitive": false,
  "maxResults": 50,
  "regexTimeoutMs": 200
}
```

**queryMode**: `literal` (default, fastest) | `wildcard` | `regex`

**Never use** `text_search` to find types/methods/symbols — use `find_references`.

---

## Pre-commit checklist

```
☐ get_errors              — no compilation errors
☐ get_warnings            — no critical warnings
☐ find_unused_code        — no dead code
☐ find_async_issues       — no async/await problems
☐ find_performance_issues — no perf anti-patterns
☐ get_code_metrics        — complexity acceptable
```

---

## Pagination

Default `pageSize=5`. For tools that support pagination (`find_references`, `find_callers`, `find_callees`, `find_attribute_usages`, `get_errors`, `get_warnings`, `find_async_issues`, `find_performance_issues`, `find_unused_code`):

```json
{ "page": 0, "pageSize": 20 }  // first 20 results
{ "page": 1, "pageSize": 20 }  // next 20 results
```

Max `pageSize`: 200.

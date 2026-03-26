---
name: roslyn-cs
description: OOP C# programming via cs tool — 86 actions. Types, members, AND statement-level CRUD inside method bodies at any nesting depth. Block path navigation, expression replacement, wrap/unwrap, extract method, inline variable. Pre-write syntax validation.
user_invocable: true
trigger: ANY C# code manipulation — create/add/update/delete types, members, statements. Insert/replace/delete inside if/for/try/switch blocks. Read block tree. Expression editing. Wrap/unwrap statements.
no_trigger: Read-only code analysis without modifications (use roslyn-code), non-C# files, VS IDE operations.
---

# CS Tool — OOP C# Programming

> `cs` is a KNOWN MCP tool — call it directly: `cs action:"create_class" name:"Foo"`. Do NOT use `search_tools` or `get_tool_schema` to find it. All 87 actions are listed below.
> Every mutation auto-reloads the file and returns `{ success, action, filePath, diagnostics[] }`. No `reload_file`, `validate_text`, or `search_tools` needed after cs operations.
>
> **MANDATORY RULES:**
> 1. **PARALLELISM FIRST:** If operations are independent — send ALL tool calls in a single message. Creating 5 classes? 5 parallel `cs batch` calls in ONE message. Never drip-feed 2-4 at a time. Plan upfront, execute at once.
> 2. NEVER use Write, Edit, or Read for .cs files. ONLY `cs` tool. If cs returns error — fix params and retry, never fall back.
> 3. After EVERY edit: `vs action:"open_file"` + `vs action:"goto_line"` so user sees the change.
> 4. Constructor = `add_constructor`, NOT `add_method`.

## Parameters

| Param | Type | Used by |
|-------|------|--------|
| `action` | string, required | ALL — selects operation |
| `target` | string | Most — `TypeName` or `TypeName.MemberName` or block path `TypeName.Method.if[0].try[0]` |
| `name` | string | Name of new element, or statement index, or block kind |
| `body` | string | C# code: method body, statement, expression |
| `returnType` | string | Return type, field type, exception type |
| `modifiers` | string | `public`, `private static`, `public async virtual` |
| `parameters` | string | `int x, string name` |
| `value` | string | Default value, position (`start`/`end`/`after:N`/`before:N`), wrapper type |
| `filePath` | string | File path (for create or partial class targeting) |
| `namespace` | string | Namespace for create actions |
| `baseTypes` | string | Base types: `BaseClass, IDisposable` |
| `accessors` | string | Property: `get; set;` or `get; private set;` |

## Actions Reference

### Create
| Action | Required | Optional |
|--------|----------|----------|
| `create_class` | `name` | `namespace`, `filePath`, `modifiers`, `baseTypes` |
| `create_interface` | `name` | `namespace`, `filePath` |
| `create_struct` | `name` | `namespace`, `filePath` |
| `create_record` | `name` | `namespace`, `filePath` |
| `create_enum` | `name` | `namespace`, `filePath` |
| `create_delegate` | `name`, `returnType`, `parameters` | `namespace`, `filePath` |
| `create_partial` | `target` (existing type), `filePath` | — |

### Add members

| Action | Required | Optional |
|--------|----------|----------|
| `add_method` | `target` (type), `name`+`returnType` OR `body` (full decl) | `modifiers`, `parameters`, `body` |
| `add_constructor` | `target` (type), `body` | `parameters` |
| `add_field` | `target`, `name`, `returnType` OR `body` (batch) | `modifiers`, `value` |
| `add_property` | `target`, `name`, `returnType` OR `body` (batch) | `modifiers`, `accessors`, `value` |
| `add_event` | `target`, `name` | `returnType` (event type, default: EventHandler) |
| `add_enum_member` | `target` (enum), `name` | `value` |
| `add_parameter` | `target` (Type.Method), `parameters` | — |
| `add_attribute` | `target`, `name` (attribute) | — |
| `add_using` | `target`, `name` (one or comma-separated) | — |
| `add_base_type` | `target`, `name` (base type) | — |
| `add_interface` | `target`, `name` (interface) | — (adds + generates stubs) |
| `add_case` | `target` (Type.Method), `name` (label), `body` | — |
| `add_statement` | `target` (Type.Method), `body` | `value` (start/end/index) |

**Batch mode:** `add_field`, `add_method`, `add_property` accept `body` with multiple C# declarations. Skip `name`/`returnType` to trigger batch.
`add_using` accepts comma-separated: `name:"System.IO, System.Linq, System.Windows.Forms"`

**Interfaces:** `create_interface` + `add_method` with body for signatures.
Example: `cs action:"add_method" target:"IMyInterface" body:"void Save(string name);\nstring Load(string name);"`

### Update

| Action | Required | Optional |
|--------|----------|----------|
| `update_method` | `target` (Type.Method or Type.Method(sig)), `body` | — |
| `update_constructor` | `target` (Type or Type.ctor(sig)), `body` | `parameters` |
| `update_property` | `target` (Type.Prop) | `accessors`, `body`, `value` |
| `update_block` | `target` (Type.Method), `name` (index/kind), `body` | — |
| `rename` | `target` (Type.Member), `name` (new name) | — |
| `change_type` | `target` (Type.Member), `returnType` | — |
| `change_modifiers` | `target`, `modifiers` | — |
| `change_accessibility` | `target`, `modifiers` | — |

**update_property:** `body` → expression body (`=> expr;`). `value` → initializer (`= expr;`). `accessors` → accessor list.

### Delete
| Action | Required |
|--------|----------|
| `delete_method` | `target` (Type.Method) |
| `delete_constructor` | `target` (Type.ctor or Type.ctor(sig)) |
| `delete_field` | `target` (Type.Field) |
| `delete_property` | `target` (Type.Property) |
| `delete_event` | `target` (Type.EventName) |
| `delete_class` | `target` |
| `delete_attribute` | `target`, `name` |
| `delete_block` | `target` (Type.Method), `name` (index/kind) |

### Move
| Action | Required |
|--------|----------|
| `move_method` | `target` (Type.Method), `name` (after which member) |
| `move_to_file` | `target` (TypeName) |
| `insert_before` | `target` (Type.AnchorMember), `body` |
| `insert_after` | `target` (Type.AnchorMember), `body` |

### Generate
| Action | Required |
|--------|----------|
| `implement_interface` | `target` (class), `name` (interface) |
| `generate_constructor` | `target` (class) — from readonly fields |
| `generate_equals` | `target` (class) |
| `generate_tostring` | `target` (class) |
| `from_symbol` | `target` |

### Read
| Action | Required | Returns |
|--------|----------|--------|
| `tree` | `target` (Type or Type.Method) | Class: members list. Method: recursive block tree with `{ kind, index, line, path, summary, children[] }` |
| `method_body` | `target` (Type.Method) | Source code + indexed statements |
| `block_body` | `target` (block path) | Statements at any nesting depth |
| `hierarchy` | `target` | Base types, interfaces |
| `accessibility` | `target` | Modifiers |
| `parameters` | `target` (Type.Method) | Parameter details |
| `attributes` | `target` | Attribute list |
| `usings` | `target` | Using directives |
| `accessors` | `target` (Type.Property) | Get/set details |
| `switch_sections` | `target` (Type.Method) | Case blocks |

### Validate & Format
| Action | Required |
|--------|----------|
| `diagnostics` | `target` or `filePath` |
| `format` | `target` or `filePath` |
| `simplify` | `target` or `filePath` |
| `data_flow` | `target` (Type.Method) |
| `symbol_info` | `target` |
| `type_info` | `target` |

### Statement-Level CRUD (use block path in target)
| Action | Required | Description |
|--------|----------|-------------|
| `insert_statement` | `target` (path), `body` | `value`: start/end/before:N/after:N |
| `update_statement` | `target` (path), `name` (index), `body` | Replace one statement |
| `delete_statement` | `target` (path), `name` (index) | Delete one statement |
| `wrap_statements` | `target` (path), `name` (range like `2-4`), `value` (if/try/using/for/while/lock) | `body`: catch body for try |
| `unwrap_block` | `target` (path), `name` (index) | Remove wrapper, keep contents |
| `replace_expression` | `target` (path), `name` (index), `body` | `value`: condition/right/left |
| `add_catch` | `target` (path), `name` (try index) | `returnType`: exception type, `parameters`: var name. `value`:"finally" for finally |
| `add_else` | `target` (path), `name` (if index), `body` | `value`: condition for else-if |
| `extract_method` | `target` (path), `name` (new method name), `value` (range) | `returnType`, `modifiers`, `parameters` |
| `inline_variable` | `target` (path), `name` (variable name) | Replaces all usages, deletes declaration |

### Expression Builders

All: `target` = Type.Method (or block path). Actions: `expr_if`, `expr_switch`, `expr_try`, `expr_using`, `expr_lock`, `expr_for`, `expr_while`, `expr_foreach`, `expr_return`, `expr_throw`, `expr_var`, `expr_await`, `expr_new`, `expr_lambda`, `expr_invoke`. Use `get_tool_schema` for parameter details.

## Block Path

`target` for statement-level actions is a dot-separated path:

```
Type.Method                      → method body
Type.Method.if[0]                → first if
Type.Method.if[0].else           → else clause
Type.Method.try[0].catch[0]      → first catch
Type.Method.try[0].finally       → finally
Type.Method.for[0] / foreach[0] / while[0] / using[0] / lock[0]
Type.Method.switch[0].case[1]    → second case
```

## Batch Mode (v1.18.11+)

`action: "batch"` — execute multiple cs actions in ONE call. **~6-7x speedup.**

```json
{"action": "batch", "actions": [
  {"action": "create_class", "name": "MyService", "namespace": "MyApp", "baseTypes": "IMyService"},
  {"action": "add_field", "target": "MyService", "name": "_repo", "returnType": "IRepository", "modifiers": "private readonly"},
  {"action": "add_constructor", "target": "MyService", "parameters": "IRepository repo", "body": "_repo = repo;"},
  {"action": "add_method", "target": "MyService", "name": "GetAll", "returnType": "List<Item>", "modifiers": "public", "body": "return _repo.GetAll();"},
  {"action": "add_using", "target": "MyService", "name": "System.Collections.Generic"}
]}
```

**Rules:** sequential execution, stops on first error, diagnostics once at end, no nested batch.

**Use batch for:** create + fill a type, add multiple members, any scaffolding with known actions upfront.
**Don't use batch for:** single operation, operations needing intermediate diagnostics, read-only actions.

### Universal Batch (`batch` tool)

Combine ANY tools in one call. All independent tool calls (cs, vs, reload_file, find_references, etc.) can run in a single batch — never call them one-by-one:

```json
{"tool": "batch", "args": {"actions": [
  {"tool": "cs", "args": {"action": "batch", "actions": [...]}},
  {"tool": "vs", "args": {"action": "open_file", "target": "Foo.cs"}}
]}}
```

## Workflow: Build a class

**ALWAYS use `cs batch`.** Never add members one-by-one. One batch = one class:

```
cs action:"batch" actions:[
  {"action":"create_class", "name":"OrderService", "namespace":"MyApp"},
  {"action":"add_field", "target":"OrderService", "name":"_repo", "returnType":"IOrderRepo", "modifiers":"private readonly"},
  {"action":"add_constructor", "target":"OrderService", "parameters":"IOrderRepo repo", "body":"_repo = repo;"},
  {"action":"add_method", "target":"OrderService", "name":"GetAll", "returnType":"List<Order>", "modifiers":"public", "body":"return _repo.GetAll();"},
  {"action":"add_using", "target":"OrderService", "name":"System.Collections.Generic"}
]
```
Then: `vs action:"open_file"` → `cs action:"tree"` to verify.

**Multiple independent classes/interfaces?** Send parallel `cs batch` calls — one per type — ALL in a single message. Example: creating IOrderRepo + OrderService + OrderDto = 3 parallel cs batch calls in ONE turn, not 3 turns.

**ANTI-PATTERN:** Do NOT create types one-by-one across multiple turns. Do NOT add members with individual cs calls when batch exists.

## Workflow: Edit inside a method

1. `cs action:"tree" target:"Type.Method"` — get block tree with paths
2. `cs action:"update_statement" target:"Type.Method" name:"0" body:"..."`
3. `vs action:"open_file"` — show to user

## Overload Resolution

```
target: "Class.Method"              → single method (error if multiple overloads)
target: "Class.Method(int,string)"  → specific overload by param types
target: "Class.ctor(string)"        → specific constructor overload
target: "Class.ctor()"              → parameterless constructor
```

- Read ops with multiple overloads → returns **array** with `hint` field. Use `hint` for follow-up calls.
- Write ops with multiple overloads → **error** with list of signatures. Must specify exact one.
- Match by type name only: `Process(string)` not `Process(string input)`. Short names work.

## Error Recovery

| Error | Fix |
|-------|-----|
| `target must be TypeName.MethodName` | Use `Type.Method` format |
| `Method 'X' not found` | `cs action:"tree" target:"Type"` to see members |
| `Type 'X' not found` | `get_solution_structure` to find types |
| `Invalid C# syntax` | Fix code in `body` and retry |
| `block not found` | `cs action:"tree" target:"Type.Method"` to see paths |

## Known Gotchas

1. **Non-block if** — `if (x) return;` without braces cannot have statements inserted. Use `tree` to check.
2. **add_else index** — `name` counts only if-statements (0-based), not all statements.
3. **Partial classes (WinForms)** — `add_field`/`add_method` may go to `.Designer.cs`. Use `filePath` to target.
4. **validate_text** requires `filePath` parameter.
5. **Overloaded methods** — without signature, read ops return array, write ops error.
6. **update_property body vs value** — `body` = expression body (`=>`), `value` = initializer (`=`).
7. **Large generic bodies** — `<T>` in large `insert_after` body breaks JSON. Split into separate `add_method` calls or `cs batch`.

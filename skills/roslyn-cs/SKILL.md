---
name: roslyn-cs
description: OOP C# programming via cs tool — 86 actions. Types, members, AND statement-level CRUD inside method bodies at any nesting depth. Block path navigation, expression replacement, wrap/unwrap, extract method, inline variable. Pre-write syntax validation.
user_invocable: true
trigger: ANY C# code manipulation — create/add/update/delete types, members, statements. Insert/replace/delete inside if/for/try/switch blocks. Read block tree. Expression editing. Wrap/unwrap statements.
no_trigger: Read-only code analysis without modifications (use roslyn-code), non-C# files, VS IDE operations.
---

# CS Tool — OOP C# Programming


> Tool: `cs`. Single endpoint, `action` parameter selects operation. 86 actions.
> NEVER use Edit/Write/Read for .cs files. ONLY `cs` tool.
> Every mutation returns `{ success, action, filePath, diagnostics[] }` — auto-validated, no need for `validate_text` before cs tool operations.
>
> **MANDATORY RULES:**
> 1. NEVER use Write, Edit, or Read tools on .cs files. Use ONLY cs tool. NO EXCEPTIONS.
> 2. After EVERY cs tool edit, run `vs action:"open_file" target:"<filePath>"` + `vs action:"goto_line" target:"<filePath>" options:{"line": <N>}` so the user sees the change.
> 3. Constructor = `add_constructor`, NOT `add_method`. `tree target:"Form1"` = class tree. `tree target:"Form1.MethodName"` = method tree. Do NOT use `Form1.Form1` for constructor.
> 4. If cs tool returns error, READ the error, FIX your parameters, RETRY. Never fall back to Write/Edit.

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

### Utility
| Action | Required | Description |
|--------|----------|-------------|
| `mode` | `name` (`dev` or `normal`) | Dev mode: skip format and diagnostics for faster bulk writes. Normal: full validation. |

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

**Batch mode:** `add_field`, `add_method`, `add_property` accept `body` with multiple C# declarations. Roslyn parses all, adds in 1 operation. Skip `name`/`returnType` to trigger batch.
`add_using` accepts comma-separated: `name:"System.IO, System.Linq, System.Windows.Forms"`

**Interfaces:** Use `create_interface` + `add_method` with body for signatures. NEVER use Write for interfaces.
Example: `cs action:"add_method" target:"IMyInterface" body:"void Save(string name);\nstring Load(string name);\nbool Delete(string name);"`

**Dev mode:** `cs action:"mode" name:"dev"` — skip format and diagnostics, faster writes. `cs action:"mode" name:"normal"` — restore full validation. Use dev mode for bulk creation, switch to normal before build/debug.


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

**update_property parameters:**
- `body` — converts to expression body (`=> expr;`), removes accessor list and initializer
- `value` — updates initializer (`= expr;`), keeps accessor list intact
- `accessors` — updates accessor list (`get; set;` or `get; private set;`)


### Delete
| Action | Required |
|--------|----------|
| `delete_method` | `target` (Type.Method) |
| `delete_constructor` | `target` (Type) |
| `delete_field` | `target` (Type.Field) |
| `delete_property` | `target` (Type.Property) |
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

### Expression Builders (add structured statement via SyntaxFactory)

All: `target` = Type.Method (or block path). Statement appended to end (or use `accessors` for position).

| Action | Key params |
|--------|----------|
| `expr_if` | `name` (condition), `body` |
| `expr_switch` | `name` (expression), `body` (case statements) |
| `expr_try` | `body` (try code), `value` (catch code) |
| `expr_using` | `name` (var decl), `value` (initializer), `body` |
| `expr_lock` | `name` (lock object), `body` |
| `expr_for` | `name` (init), `value` (condition), `parameters` (increment), `body` |
| `expr_while` | `name` (condition), `body` |
| `expr_foreach` | `name` (variable), `value` (collection), `body` |
| `expr_return` | `body` or `value` (expression) |
| `expr_throw` | `name` (exception type), `value` (message) |
| `expr_var` | `name` (variable), `returnType` (type), `value` (initializer) |
| `expr_await` | `body` or `value` (async expression) |
| `expr_new` | `name` (variable), `returnType` (type), `value` (constructor args) |
| `expr_lambda` | `name` (variable), `parameters`, `body` |
| `expr_invoke` | `name` (method name), `value` (arguments) |

## Block Path

`target` for statement-level actions is a dot-separated path:

```
Type.Method                      → method body
Type.Method.if[0]                → first if
Type.Method.if[0].else           → else clause
Type.Method.try[0]               → try body
Type.Method.try[0].catch[0]      → first catch
Type.Method.try[0].finally       → finally
Type.Method.for[0]               → for body
Type.Method.foreach[0]           → foreach body
Type.Method.while[0]             → while body
Type.Method.using[0]             → using body
Type.Method.lock[0]              → lock body
Type.Method.switch[0].case[1]    → second case
```

## Error Recovery

| Error | Fix |
|-------|-----|
| `target must be TypeName.MethodName` | Use `Type.Method` format |
| `Method 'X' not found` | `cs action:"tree" target:"Type"` to see members |
| `Type 'X' not found` | `get_solution_structure` to find types |
| `Invalid C# syntax` | Fix code in `body` and retry |
| `block not found` | `cs action:"tree" target:"Type.Method"` to see paths |

**NEVER fall back to Edit/Write. Fix the call and retry.**

## Workflow: How to build a class from scratch


**Fast (batch):**
1. `cs action:"create_class" name:"MyClass" namespace:"MyApp" filePath:"."`
2. `cs action:"add_field" target:"MyClass" body:"private string _name;\nprivate int _count;\npublic MyClass(string name) { _name = name; }\npublic void DoWork() { Console.WriteLine(_name); }"`
3. `cs action:"add_using" target:"MyClass" name:"System.IO, System.Linq"`
4. `vs action:"open_file" target:"MyClass.cs"` <- show to user!

**Step-by-step (when you need control):**
1. `cs action:"create_class" name:"MyClass" namespace:"MyApp" filePath:"."`
2. `cs action:"add_field" target:"MyClass" name:"_name" returnType:"string" modifiers:"private"`
3. `cs action:"add_property" target:"MyClass" name:"Name" returnType:"string" modifiers:"public" accessors:"get; set;"`
4. `cs action:"add_constructor" target:"MyClass" parameters:"string name" body:"_name = name;"`  <- NOT add_method!
5. `cs action:"add_method" target:"MyClass" name:"DoWork" returnType:"void" modifiers:"public" body:"Console.WriteLine(_name);"`
6. `cs action:"add_using" target:"MyClass" name:"System.IO"`
7. `vs action:"open_file" target:"MyClass.cs"` <- show to user!
8. `cs action:"tree" target:"MyClass"` <- verify result


## Workflow: How to edit inside a method

1. `cs action:"tree" target:"MyClass.DoWork"` <- get recursive tree with indices and paths
2. Pick the path and index you need
3. `cs action:"update_statement" target:"MyClass.DoWork" name:"0" body:"Console.WriteLine(\"updated\");"`
4. Or for nested: `cs action:"insert_statement" target:"MyClass.DoWork.if[0]" body:"Log();" value:"start"`
5. `vs action:"open_file" target:"MyClass.cs"` <- show to user!
6. `cs action:"tree" target:"MyClass.DoWork"` <- verify result

## Workflow: How to handle WinForms

- Constructor body should call `InitializeComponent();` first, then your init code
- Event handlers: `add_method` with correct signature, e.g. `parameters:"object? sender, EventArgs e"`
- Add handler in constructor/init: `_button.Click += Button_Click;` via `add_statement` or in constructor body
- UI setup: create controls and set properties in an init method, add to Controls collection

## Constructor CRUD


Constructors are NOT methods. They use separate actions:

| Action | Usage |
|--------|-------|
| `add_constructor` | `target:"MyClass" body:"InitializeComponent();" parameters:"string name"` |
| `update_constructor` | `target:"MyClass.ctor(string)" body:"Name = name;"` — updates specific overload |
| `delete_constructor` | `target:"MyClass.ctor()"` — deletes specific overload |

- `add_constructor` adds a NEW constructor. If one exists, you get CS0111 duplicate error.
- To change existing constructor: use `update_constructor`, NOT delete+add.
- Target format: `"MyClass"`, `"MyClass.ctor"`, or `"MyClass.ctor(string,int)"` for specific overload.
- `method_body`, `tree`, `block_body`, `add_statement` all work with `ctor` target.
- Example: `cs action:"method_body" target:"MyClass.ctor"` — returns all constructor overloads.
- Example: `cs action:"add_statement" target:"MyClass.ctor(string)" body:"Console.WriteLine(name);"` — adds to specific constructor.



## Overload Resolution (v1.18.8+)


When a class has multiple methods/constructors with the same name, use signature in target:

```
target: "Class.Method"              → single method (error if multiple overloads)
target: "Class.Method(int,string)"  → specific overload by param types
target: "Class.ctor"                → single constructor (error if multiple)
target: "Class.ctor(string)"        → specific constructor overload
target: "Class.ctor()"              → parameterless constructor
```

### Read operations (method_body, tree, parameters)
- Multiple overloads without signature → returns **array** of all overloads with `hint` field
- Each overload includes `filePath` (important for partial classes)
- Use `hint` value as target for follow-up calls

### Write operations (update_method, delete_method, update_constructor)
- Multiple overloads without signature → **error** with list of available signatures
- Must specify exact signature to update/delete

### Partial classes
- Overloads across different partial files are found automatically
- Each result includes `filePath` showing which file contains that overload
- Example: `Drive()` in Car.cs, `Drive(int)` in Car.Engine.cs — both found

### Parameter matching
- Match by type name only (not parameter names): `Process(string)` not `Process(string input)`
- Short names work: `FolderInfo` matches `Graph.FolderInfo`
- Comma-separated, no spaces needed: `(int,string)` or `(int, string)`


## Known Gotchas


1. **Non-block if** — `if (x) return;` without braces cannot have statements inserted. Use `tree` to check if node has `children[]` before inserting.
2. **add_else index** — `name` counts only if-statements (0-based), not all statements. Use `tree` to find the right if-index.
3. **Partial classes (WinForms)** — `add_field`/`add_method` may go to `.Designer.cs` instead of `Form1.cs`. Use `filePath` parameter to target specific file.
4. **validate_text** requires `filePath` parameter — always include it.
5. **Overloaded methods** — without signature, read ops return array, write ops error. Always check hint field and use signature for writes.
6. **update_property body vs value** — `body` converts to expression body (`=>`), `value` updates initializer (`=`). Use `value` for `{ get; set; } = "default"` properties.


## Key Principle: Always Verify

After EVERY mutation, verify with `tree` or `method_body` that the code is correct.
Do NOT trust success response alone — read back and confirm.

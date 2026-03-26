# Changelog

## [1.18.10] - 2026-03-26


### Fixed
- `update_statement` now supports replacing one statement with multiple statements
- `change_type` now supports changing method/constructor parameter types via `Class.Method.paramName` syntax
- `update_method` no longer misdetects method body as full declaration when body calls same-named method


## [1.18.9] - 2026-03-25

### Fixed
- `update_method` — body containing method name (e.g. `return GetAccessToken(false)`) no longer misdetected as full declaration
- Events (`EventFieldDeclarationSyntax`) now visible in `cs tree` output
- Added `EnumMember` and `Delegate` to `cs tree` output
- `get_type_members` — returns disambiguation when multiple types share same name in different namespaces

### Added
- `delete_event` action with partial class support
- `add_event` — documented `returnType` parameter for custom event types (e.g. `EventHandler<User>`)

## [1.18.8] - 2026-03-25

### Added
- **Overload resolution** — all cs tool actions now support method/constructor overload selection via signature in target: `Class.Method(int,string)`, `Class.ctor(string)`
- **Partial class support** — methods, properties, fields, events found across ALL partial declarations automatically
- **Constructor support in cs tool** — `method_body`, `tree`, `block_body`, `add_statement`, `data_flow` all work with `ctor` target
- **`update_property` value parameter** — update property initializer (`= "default"`) without converting to expression body
- **`extract_method` auto-parameters** — automatically detects required parameters via DataFlow analysis
- **`understand_method` / `get_method_body` overloads** — general tools return all overloads when multiple exist

### Fixed
- `method_body` with target `Class.ctor` — was "Method 'ctor' not found", now works
- `update_constructor` / `delete_constructor` — were always targeting first constructor, now support signature selection
- `update_property` with `body` — was leaving old initializer (`=> expr = oldValue`), now clears it
- `accessors` on partial class property — was "Property not found", now searches all partials
- `insert_before` / `insert_after` on partial class member — was "Anchor not found", now searches all partials
- `change_modifiers` / `change_type` — now support overload signature and partial class members
- `add_attribute` / `delete_attribute` / `attributes` — now search across partial declarations
- `accessibility` — now finds members in any partial file

### Changed
- All 20+ cs tool methods refactored to use unified helpers: `ParseTargetWithSignature`, `CsFindMethodOrConstructor`, `CsFindMethodAcrossPartialsAsync`, `CsFindMemberAcrossPartialsAsync`
- Read operations (method_body, tree, parameters) return array of all overloads with `hint` field for precise targeting
- Write operations (update_method, delete_method) require signature when multiple overloads exist, error message lists available signatures
- `HandleGetMethodBodyAsync` / `HandleUnderstandMethodAsync` return `Task<object>` to support both single result and overloads array

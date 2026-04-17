# Changelog



## [1.18.15] - 2026-04-17


### Added
- **`diag` tool — ClrMD-based memory/heap diagnostics (25 actions)**: programmatic leak hunting for any live .NET process (Framework 4.x, Core, .NET 5+).
  - **Snapshot lifecycle (async + polling)**: `snapshot_create` (MiniDumpWriteDump with FullMemory, returns immediately with `status:"writing"`), `snapshot_info` (poll for ready/failed/writing), `snapshot_list`, `snapshot_rename`, `snapshot_delete`, `snapshot_cleanup` (TTL-based)
  - **Process discovery**: `find_process` with 5 matching strategies (pid / processName / assemblyName / windowTitle / typeName). Permissive: accepts any combination.
  - **Heap analysis**: `type_stats` (top N by size/count/name with pagination + filter), `instance_count`, `compare_snapshots` (diff by type with added/removed/grew/shrunk + hints), `find_roots` (reverse ref map + BFS to GC root, multi-level retention chains)
  - **Object inspection**: `object_info` (fields with values), `string_value`, `array_preview`, `event_subscriptions` (delegate/EventHandler field walk)
  - **GC diagnostics**: `gc_stats` (Gen0/1/2/LOH/POH/Frozen breakdown), `finalizer_queue` (top types waiting for finalization)
  - **WinForms-specific**: `live_forms`, `live_controls` (walks type hierarchy looking for System.Windows.Forms.Form/Control)
  - **Advanced**: `static_holders` (static root references to a type), `trace_object` (cross-snapshot identity via Handle/field-fingerprint), `leak_hunt` (guided workflow), `cef_state` (CefSharp ChromiumWebBrowser inventory + pinned StrongHandles + `Cef.IsInitialized`), `dispose_orphans` (heuristic scan for IDisposable with disposed parent — catch-all for dispose-chain bugs), `force_gc` (composite: attach + Immediate `GC.Collect(2,Forced)` + detach — eliminates GC-lag ghosts before compare)
  - **Cross-process snapshot safety**: `compare_snapshots` and `trace_object` refuse differing PIDs; `snapshot_list` surfaces `distinctPids` with warning
  - **Architecture**: snapshots persisted under `.roslyn-mcp/snapshots/` (`.dmp` + `.meta.json`), all work done server-side, AI receives only aggregated/paginated/hinted JSON
  - **NuGet**: `Microsoft.Diagnostics.Runtime` 3.1.456901 (netstandard2.0)
  - Skill: `/roslyn-diag` for workflow guidance
- **`vs` tool — new debug actions**:
  - `attach_to_process` — programmatic attach by PID via DTE (no UI dialog). Options: `{break: true}` to break immediately after attach.
  - `execute_immediate` — run arbitrary C# statement in the Immediate Window while debugger is in Break mode. Enables `GC.Collect`, expression evaluation, state inspection from AI.
  - Force-GC pattern documented in `roslyn-vs` and `roslyn-diag` skills.
- **Auto-cleanup of snapshot dumps on VS shutdown**: `.dmp` files in `<solutionDir>/.roslyn-mcp/snapshots/` are automatically deleted when Visual Studio closes (`.meta.json` kept for history). Prevents multi-GB accumulation across sessions.



## [1.18.14] - 2026-04-01

### Added

- **MD Index**: Multi-file markdown indexing with hybrid search (FTS5 + ONNX embeddings)
  - `md action:"index"` — on-demand indexing of all .md files in a directory (recursive)
  - `md action:"search_all"` — hybrid search: FTS5 keyword matching + semantic vector search (ONNX embeddings)
  - `md action:"stats"` — index statistics (files, sections, size)
  - Hierarchical section tree with `treePath` (e.g. `Doc/Chapter/Section`)
  - On-demand: indexes only changed files (lastModified check), cleans up deleted files
  - Auto-reindex: editing via md tool automatically updates index (FTS + vectors)
  - Uses `winsqlite3.dll` (Windows built-in) for FTS5 support in VSIX context
  - Excludes `bin/`, `obj/`, `node_modules/`, `.git/`, `.vs/` from indexing
  - Robust query sanitization: handles any input (versions, URLs, special chars, any language)
- **MD Tree-first creation**: structured document creation without thinking about `#` levels
  - `md action:"create_doc"` — create new .md with title
  - `md action:"add_node"` — add section under parent, auto heading level
  - `md action:"bulk_create"` — create entire document from indented tree spec in one call
  - `md action:"tree"` — hierarchical tree view with children[] and treePath
  - `md action:"rename_node"` — rename section heading
  - `md action:"move_node"` — move section to different parent, auto-adjusts heading levels
- **SQLite migration**: Added `Microsoft.Data.Sqlite.Core` + `SQLitePCLRaw.provider.winsqlite3` for FTS5
  - `System.Data.SQLite` remains for existing DevGraphDb/TestCaseService (no breaking changes)
  - `AssemblyResolve` handler for VSIX native DLL loading

### Fixed
- `vs_query what:"errors"` now returns only errors (was returning errors + warnings mixed)
- `vs_query what:"warnings"` now returns only warnings

## [1.18.13] - 2026-04-01

### Added
- **BlazorPilot** — new `blazor` tool with 35 actions for Blazor & web UI automation via Chrome DevTools Protocol (CDP). Full DevTools access with Roslyn C# integration.
  - **Roslyn analysis:** `inspect` parses .razor files (components, events, @bind, state, methods with file:line), `list_pages` finds all Blazor routes
  - **UI actions:** `click`, `click_by_text`, `type`, `press_key`, `select`, `select_by_text`, `hover`, `scroll`, `check/uncheck`, `wait`, `focus`, `clear` — real CDP Input clicks that work with Blazor EventCallbacks
  - **Page observation:** `accessibility_tree` (with HTML snippets for interactive elements), `snapshot`, `screenshot` (full page or element)
  - **DevTools:** `console` logs, `network` requests with status codes and errors, `cookies`, `storage` (localStorage/sessionStorage)
  - **CSS & inspection:** `css` (get/set computed styles), `element_state` (visible, enabled, rect, attrs), `highlight` (visual red overlay), `count` elements
  - **Emulation:** `viewport` (mobile/tablet responsive testing), `emulate` (dark/light mode), `performance` metrics
  - **Smart features:** `click_by_text` searches overlays/modals first; `select` auto-detects value vs text match; auto-CDP port configuration in launchSettings.json
  - Works with **Blazor Desktop** (WebView2/MAUI), **Blazor Server/WASM**, and **any website** in Chrome/Edge


### Fixed

- **find_references** — added `containingType` parameter for precise symbol resolution. Previously, searching for a common property name (e.g. `Name`) returned references from ALL types with that member. Now you can filter: `find_references symbolName:"Name" containingType:"Animal"` to get only references to `Animal.Name`. Matches ReSharper "Find Usages" precision.

## [1.18.12] - 2026-03-27
 
### Added
- **`cs action:"create_file"`** — create .cs file from full source code in one call. Validates syntax via Roslyn, formats, saves, and indexes in workspace. Fastest way to scaffold new files with compile-time safety.
- **cs tool fallback** — unknown cs actions (e.g. `get_type_info`) automatically redirect to matching Roslyn tools instead of returning error.

### Fixed
- **`create_class`/`create_type` overwrite protection** — previously, calling `create_class` with a `filePath` pointing to an existing file would silently overwrite it, destroying the original code. Now returns a clear error: "File already exists. Use add_method/add_field to modify the existing type."
- **JSON parser resilience** — when LLM sends cs tool arguments as a string (instead of JSON object), it sometimes appends XML tags like `</invoke>` at the end, breaking JSON parsing. Now `ParseCsArgs` trims any trailing content after the last `}` before parsing.
- **VS instance switching** — `switch_instance` with direct port now skips full scan (instant, no hang). VS instances without a solution are now visible in `list_instances` without opening the Dashboard. No-solution VS supports `vs`/`vs_query` tools to open a solution via `open_solution`.
- **`md` tool `insert_section`** — fixed insertion position: section now inserts exactly where specified (`after` heading), not at wrong location.

## [1.18.11] - 2026-03-26

### Added
- **Universal `batch` tool** — execute multiple tools (cs, vs, reload_file, find_references, etc.) in ONE call. Stops on first error, returns results array.
- **`cs action:"batch"`** — execute multiple cs actions in one call (create class + add fields/methods/usings). Auto-indexes new files via `EnsureNewFileInWorkspaceAsync`.
- **`[JsonExtensionData]` on CsArgs** — unknown JSON properties no longer break parsing. Large method bodies parse correctly.

### Fixed
- **Proxy: VS without solution** — now shows VS instances even when no solution is open.
- New files created by `cs create_*` immediately visible to subsequent actions.

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




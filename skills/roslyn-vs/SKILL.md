---
name: roslyn-vs
description: "Full Visual Studio IDE control — build, debug, breakpoints, files, find/replace, deploy, configurations, self-development loop.\nTRIGGER when: ANY Visual Studio IDE operation — build, debug, breakpoints, step, deploy, find/replace, query errors/locals/callstack.\nDO NOT TRIGGER when: only analyzing C# code without IDE interaction."
---

# Visual Studio IDE Operator — Complete Reference

> Two MCP tools: `vs` (actions) and `vs_query` (queries). Always-loaded, available immediately.

---

## Knowledge Persistence Protocol

After completing a successful IDE workflow (build fix, debug session, deploy):
1. **Record the recipe** in KB via `kb_add` (type: Snippet, category: tech) if it's reusable
2. **Record lessons** (gotchas, workarounds) via `memory_remember` (Lesson, global, importance >= 7)
3. **Search KB first** before starting: `kb_search query:"deploy VSIX"` — a recipe may exist
4. **Don't record**: bug fixes (in git), session-specific context (use session memory)

---

## vs_query — Read IDE State (22 queries)

### Solution & Projects
```json
vs_query { "what": "solution" }
// → solutionPath, isOpen, saved, projects[], startupProject, activeConfiguration, projectCount

vs_query { "what": "projects" }
// → list of all projects with name, fileName, uniqueName, kind

vs_query { "what": "project", "target": "ProjectName" }
// → detailed project info: files (up to 100), outputPath, kind

vs_query { "what": "configurations" }
// → all configurations (Debug/Release/...), activeConfiguration

vs_query { "what": "startup_project" }
// → startupProjects[]
```

### Build
```json
vs_query { "what": "build_state" }
// → buildState (NotStarted/InProgress/Done), lastBuildFailedProjects

vs_query { "what": "errors" }
// → items[{description, fileName, line, column, project}], count

vs_query { "what": "warnings" }
// → same format as errors

vs_query { "what": "output", "target": "Build", "options": {"lastLines": 50} }
// → lines from Output pane. target: "Build", "Debug", "Tests", "General", etc.
// → if pane not found, returns availablePanes[]
```

### Debug
```json
vs_query { "what": "debug_state" }
// → mode (Design/Run/Break), isDebugging, lastBreakReason,
//   currentFunction, currentFile, currentLine, breakpointCount

vs_query { "what": "callstack" }
// → frames[{functionName, language, module}], count
// ⚠️ Requires Break mode

vs_query { "what": "locals" }
// → locals[{name, value, type}], count
// ⚠️ Requires Break mode

vs_query { "what": "threads" }
// → threads[{id, name, isFrozen, isAlive}], count
// ⚠️ Requires active debug session

vs_query { "what": "expression", "target": "myVar.ToString()" }
// → expression, value, type, isValid
// ⚠️ Requires Break mode. Evaluates any C# expression.

vs_query { "what": "breakpoints" }
// → breakpoints[{file, line, column, enabled, condition, conditionType,
//   hitCountTarget, currentHits, functionName, name, tag}], count

vs_query { "what": "breakpoint_hits" }
// → lastHitBreakpoints[{file, line, functionName, currentHits}]
```

### Editor


```json
vs_query { "what": "active_document" }
// → name, fullName, path, language, saved, readOnly

vs_query { "what": "open_documents" }
// → documents[{name, fullName, saved, language}], count

vs_query { "what": "selected_items" }
// → items[{name, projectName, itemName}], count (Solution Explorer selection)

vs_query { "what": "status_bar" }
// → text

vs_query { "what": "editor_context" }
// → file, fullPath, language, cursor{line,column}, wordAtCursor,
//   currentLineText, surroundingCode, selection (if any),
//   hasBreakpoint, debugValue (if Break mode),
//   roslynContext: {namespace, className, methodName, methodSignature,
//                   methodStartLine, methodEndLine, scopeChain,
//                   symbolKind, symbolType, symbolFullName, diagnostics}

vs_query { "what": "editor_context", "options": {"mode": "method"} }
// → same as above + methodBody (full method source code)

vs_query { "what": "editor_context", "options": {"surroundingLines": 10} }
// → ±10 lines around cursor instead of default ±5

vs_query { "what": "bookmarks" }
// → bookmarks[{file, fullPath, line, label}], count (AI-set bookmarks)
```

### Bookmarks
```json
vs { "action": "bookmark_set", "target": "file.cs", "options": {"line": 42, "label": "description"} }
// Set bookmark at file:line with optional label. Toggle: call again on same line to remove.

vs { "action": "bookmark_set" }
// Set bookmark at current cursor position

vs { "action": "bookmark_next" }
// Navigate to next bookmark (all bookmarks, including manually set)

vs { "action": "bookmark_prev" }
// Navigate to previous bookmark

vs { "action": "bookmark_clear_all" }
// Remove all AI-set bookmarks (no confirmation dialog)
```



### Tests

See `/roslyn-tests` skill for full test runner documentation (run, stop, results, TDD cycle).

### Snapshot (all at once)
```json
vs_query { "what": "snapshot" }
// → { solution, buildState, debugState, activeDocument, breakpoints }
// Each section independent — one failing won't break the rest
```

---

## vs — Execute Actions (33 actions)

### Solution Management
```json
vs { "action": "set_startup", "target": "ProjectName" }
// Set startup project

vs { "action": "set_configuration", "target": "Debug" }
vs { "action": "set_configuration", "target": "Release", "options": {"platform": "x64"} }
// Switch build configuration

vs { "action": "open_solution", "target": "C:\\path\\to\\Solution.sln" }
vs { "action": "close_solution" }
vs { "action": "close_solution", "options": {"save": false} }
// Open/close solution

vs { "action": "save_all" }
// Save all files (Ctrl+S all)
```

### Build
```json
vs { "action": "build" }
// Build solution (Ctrl+Shift+B). Async — waits for completion, returns duration.

vs { "action": "build", "target": "ProjectName" }
// Build specific project only

vs { "action": "rebuild" }
// Full rebuild solution

vs { "action": "clean" }
// Clean solution
// All build actions return: { success, buildSucceeded, failedProjects, duration }
// Timeout: 5 minutes
```

### Deploy Configuration

```json
vs { "action": "enable_deploy", "target": "<ProjectName>" }
// Enable Deploy checkbox in Configuration Manager for active config
// Required for VSIX to deploy to Experimental Instance on F5

vs { "action": "disable_deploy", "target": "<ProjectName>" }
// Disable Deploy
```

### Debug — Start/Stop

```json
vs { "action": "start_debug" }
// F5 — start debugging. Default: wait=false (app needs time to load).
// Returns immediately. Use debug_monitor or vs_query to check when app is ready.

vs { "action": "start_debug", "options": {"wait": true, "timeout": 60000} }
// Wait for first breakpoint hit (up to 60s). Returns debugState when stopped.

vs { "action": "start_no_debug" }
// Ctrl+F5 — start without debugging

vs { "action": "stop_debug" }
// Shift+F5 — stop debugging

vs { "action": "restart" }
// Ctrl+Shift+F5 — restart. Default: wait=false.

vs { "action": "detach" }
// Detach debugger from process (process keeps running)
```


### Debug — Attach / Immediate


```json
vs { "action": "attach_to_process", "target": "12345" }
// Programmatic attach-by-pid — no UI dialog. target = PID as string.
// Options: {"break": true} to break immediately after attach.

vs { "action": "attach_to_process", "target": "12345", "options": { "break": true } }
// Attach + break into debugger on arrival.

vs { "action": "execute_immediate", "target": "System.GC.Collect(2, System.GCCollectionMode.Forced); System.GC.WaitForPendingFinalizers(); System.GC.Collect();" }
// Execute arbitrary C# statement in the Immediate Window.
// Debugger MUST be in Break mode — attach + break first.
// Options: {"timeout": 10000} in ms.
```

**Force-GC pattern** (useful for `diag` leak-hunt):
```
1. vs attach_to_process target:"<pid>" options:{break:true}
2. vs execute_immediate target:"System.GC.Collect(2, System.GCCollectionMode.Forced); System.GC.WaitForPendingFinalizers(); System.GC.Collect();"
3. vs continue
4. vs detach
5. diag snapshot_create pid:<pid> label:"post-gc"
```


### Debug — Stepping

> **Sync-by-default (v1.18.6+):** All stepping commands WAIT for the debugger to stop
> and return the new state in the response. No need for separate `vs_query` calls.
> The response includes `mode`, `debugState` (currentLine, currentFunction, breakpointCount).

```json
vs { "action": "step_over" }
// F10 — step over. WAITS for Break mode, returns debugState.
// Response: { success, waited, mode: "Break", debugState: {currentLine, currentFunction, ...} }

vs { "action": "step_into" }
// F11 — step into function. WAITS for Break mode.

vs { "action": "step_out" }
// Shift+F11 — step out of function. WAITS for Break mode.

vs { "action": "continue" }
// F5 from Break — continue to next breakpoint. WAITS for Break/Design mode.

vs { "action": "run_to_cursor", "target": "File.cs", "options": {"line": 42} }
// Run to specific line. WAITS for Break mode.

vs { "action": "break" }
// Pause execution (Break All). Does NOT wait.
// ⚠️ Requires Run mode
```

### options for stepping/debug commands

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `wait` | bool | **true** for step/continue/run_to_cursor, **false** for start_debug/restart | Wait for debugger to reach Break/Design |
| `timeout` | int | 30000 | Max wait in ms. Returns `timedOut: true` on expiry — never hangs |

```json
// Long-running method — increase timeout:
vs { "action": "step_over", "options": {"timeout": 120000} }

// Fire-and-forget continue:
vs { "action": "continue", "options": {"wait": false} }
```

> **Cancellation:** A new debug request automatically cancels any previous wait.
> If AI sends step_over while a previous step_over is still waiting, the old one
> returns `cancelled: true` and the new one takes over.

### Breakpoints
```json
vs { "action": "breakpoint_add", "target": "File.cs", "options": {"line": 42} }
// Add line breakpoint

vs { "action": "breakpoint_add", "target": "File.cs", "options": {"line": 42, "condition": "x > 5"} }
// Add conditional breakpoint

vs { "action": "breakpoint_add", "options": {"function": "MyClass.MyMethod"} }
// Add function breakpoint

vs { "action": "breakpoint_remove", "target": "File.cs", "options": {"line": 42} }
// Remove specific breakpoint

vs { "action": "breakpoint_toggle", "target": "File.cs", "options": {"line": 42} }
// Toggle enable/disable

vs { "action": "breakpoint_enable_all" }
vs { "action": "breakpoint_disable_all" }
vs { "action": "breakpoint_clear_all" }
// Bulk operations on all breakpoints
```

### File Operations
```json
vs { "action": "open_file", "target": "C:\\path\\to\\File.cs" }
// Open file in editor

vs { "action": "open_file", "target": "File.cs", "options": {"line": 42} }
// Open file and go to specific line

vs { "action": "close_file", "target": "File.cs" }
vs { "action": "close_file", "target": "File.cs", "options": {"save": false} }
// Close file (without target — closes active document)

vs { "action": "save_file", "target": "File.cs" }
// Save specific file (without target — saves active document)

vs { "action": "goto_line", "options": {"line": 42} }
// Go to line in active document
```

### Find & Replace
```json
vs { "action": "find", "target": "searchText" }
// Find in entire solution (Find All)

vs { "action": "find", "target": "searchText", "options": {"matchCase": true, "wholeWord": true, "filesOfType": "*.cs"} }
// Find with options

vs { "action": "replace", "target": "oldText", "options": {"replaceWith": "newText"} }
// Replace all in solution

vs { "action": "replace", "target": "oldText", "options": {"replaceWith": "newText", "matchCase": true} }
// Replace with options
```

### Universal VS Command
```json
vs { "action": "execute_command", "target": "Edit.FormatDocument" }
// Execute any Visual Studio command by name

vs { "action": "execute_command", "target": "View.SolutionExplorer" }
vs { "action": "execute_command", "target": "Edit.CommentSelection" }
vs { "action": "execute_command", "target": "Build.Cancel" }
vs { "action": "execute_command", "target": "Window.CloseAllDocuments" }
// Any VS command — the escape hatch for anything not covered above
```

---

## Instance Management

```json
list_instances
// → [{port, solutionName, solutionDirectory, extensionVersion, projectCount, connected}]

switch_instance { "solutionName": "MyApp" }
switch_instance { "port": 52851 }
switch_instance { "solutionDirectory": "C:\\Sources\\MyProject" }
// Switch proxy to different VS instance (partial match supported)
```

---

## Self-Development Loop

> Deploy is fully functional. `DeployExtension=true` in csproj, standard F5 deploys VSIX to Experimental Instance.
> Deploy checkbox must be enabled once per config: `vs action:"enable_deploy" target:"<ExtensionProject>"`

### Full cycle for VSIX extension development:

```
1. Edit code (Roslyn tools: validate_text → Edit → reload_file → get_errors)
2. vs action:"save_all"                                 → save changes
3. vs action:"build"                                    → compile + auto-deploy VSIX
4. vs_query what:"errors"                               → check errors
5. vs_query what:"output" target:"Build"                → read build log if needed
6. vs action:"enable_deploy" target:"<ExtensionProject>"  → one-time per config
7. vs action:"start_debug"                              → F5, launches Experimental Instance
8. (wait ~20s for Experimental Instance to start)
9. list_instances → switch_instance                     → connect to Exp instance
10. Test new tools there
11. switch_instance back to main instance
12. vs action:"stop_debug"                              → stop Exp instance
13. Fix bugs → goto 1
```

### Debug a specific issue:

```
1. vs action:"breakpoint_add" target:"File.cs" options:{line:42}
2. vs action:"start_debug"                    → fire-and-forget (default)
3. ... trigger the code path (click UI, etc) ...
4. Breakpoint hits → vs_query what:"debug_state"  → confirm Break mode, see line
5. vs action:"step_over"                       → WAITS, returns new line + debugState
6. vs_query what:"locals"                      → inspect variables
7. vs_query what:"expression" target:"myObj.Count"  → eval expression
8. vs action:"step_into"                       → WAITS, enters function
9. vs action:"step_out"                        → WAITS, exits function
10. vs action:"continue"                       → WAITS for next breakpoint
11. vs action:"breakpoint_clear_all"            → cleanup
12. vs action:"stop_debug"                     → done
```

> **Key difference from old workflow:** Steps 5, 8, 9, 10 now WAIT and return
> debugState automatically. No need for `vs_query what:"debug_state"` after each step.
> Only use `vs_query` when you need locals/expression/callstack — not for position.

## Known Issues & Fixes


### Phantom VS Installation
If Deploy fails with "Could not find file ... extension.vsixmanifest" errors, this is a phantom VS installation registered via vswhere but with missing files. Fix: create dummy `extension.vsixmanifest` in affected directories. Requires admin elevation.

### Build State "not available"
`vs_query what:"build_state"` returns error before the first build in a session.
This is normal — DTE BuildEvents are not populated until the first build runs.
Use `vs_query what:"snapshot"` which wraps each sub-query in try/catch.

### Deploy Checkbox Resets
The Deploy checkbox in Configuration Manager resets when switching configurations.
Always call `vs action:"enable_deploy"` after `vs action:"set_configuration"`.


## How-To: Debug a Button Click


### Step-by-step:
```
1. Find the handler method:
   cs action:"tree" target:"Form1"                    → find the click handler name

2. Set breakpoint:
   vs action:"breakpoint_add" target:"Form1.cs" options:{"line": 42}

3. Start debug:
   vs action:"start_debug"

4. Wait for app window (use UIA, not screenshot):
   screen action:"ui_find" processName:"MyApp" controlType:"Window"
   If not found, wait a few seconds and retry.

5. Click the button (UIA first, screenshot fallback):
   screen action:"ui_find" processName:"MyApp" controlType:"Button" name:"Save"
   screen action:"ui_invoke" ref:"uia_N"
   If UIA fails: take screenshot, calculate coords, mouse_click.

6. Debugger breaks — inspect state:
   vs_query what:"debug_state"    → should be "Break"
   vs_query what:"locals"         → see variable values
   vs_query what:"callstack"      → see call stack

7. Step through code:
   vs action:"step_over"          → next line
   vs_query what:"locals"         → check changed values
   vs action:"step_into"          → go into method call
   vs action:"step_out"           → return to caller

8. Continue or stop:
   vs action:"continue"           → resume execution
   vs action:"stop_debug"         → end debug session

9. Clean up:
   vs action:"breakpoint_clear_all"
```

## How-To: Build and Run App

```
1. Build:
   vs action:"build"
   vs_query what:"errors"         → check for errors

2. If errors:
   vs_query what:"errors"         → read error details
   Fix using cs tool (update_method, update_statement, etc.)
   vs action:"build"              → rebuild

3. Run without debug:
   vs action:"start_no_debug"

4. Verify app (UIA first):
   screen action:"ui_find" processName:"MyApp" controlType:"Window"
   screen action:"ui_tree" processName:"MyApp" depth:3    → see all controls
   screen action:"screenshot" target:"window" title:"MyApp" → visual check

5. Stop app:
   vs action:"stop_debug"
```

## How-To: Fix Bug Found During Debug

```
1. While stopped at breakpoint, identify the bug:
   vs_query what:"locals"         → see wrong value

2. Stop debug:
   vs action:"stop_debug"

3. Fix using cs tool (NOT Edit/Write!):
   cs action:"tree" target:"MyClass.BuggyMethod"    → find the block
   cs action:"update_statement" target:"MyClass.BuggyMethod" name:"3" body:"fixed code;"

4. Open file to show fix:
   vs action:"open_file" target:"MyClass.cs"

5. Rebuild and re-test:
   vs action:"build"
   vs action:"start_debug"        → verify fix
```



---
name: roslyn-ui-test
description: "Automated UI testing — launch apps, interact with UI elements via UIA, set breakpoints, inspect variables, verify behavior.\nTRIGGER when: end-to-end UI testing — launch app, interact with UI elements, verify behavior programmatically.\nDO NOT TRIGGER when: just screenshots or simple clicks (use roslyn-screen), or testing non-UI code."
---

# UI Automation Testing — Complete Reference

> Combines UI Automation (screen tool), Visual Studio debugging (vs/vs_query tools), and desktop automation for end-to-end application testing without manual interaction.

---

## Knowledge Persistence Protocol

After completing a successful test workflow:
1. **Record the recipe** in KB via `kb_add` (type: Snippet, category: tech)
2. **Include**: app name, UI element paths, test steps, breakpoints, assertions
3. **Tag** with app name, test type (e.g., `myapp`, `ui-test`, `login-flow`)
4. Before starting a new test, **search KB first**: `kb_search query:"<app> login test"`
5. Record **lessons learned** via `memory_remember` (Lesson, global, importance >= 7)
6. **Don't record**: bug fixes (in git), one-off debug sessions (use session memory)

---

## Core Concepts

### Element References (refs)
Every UI element returned by `ui_tree` or `ui_find` gets a cached reference like `uia_42`.
- Use refs in subsequent calls: `ui_invoke`, `ui_set_value`, `ui_get_value`, etc.
- Refs expire after **120 seconds** — re-fetch if stale
- Call `ui_clear_cache` to reset

### Control Types
Common types for filtering: `Button`, `Edit`, `ComboBox`, `CheckBox`, `RadioButton`, `MenuItem`, `TabItem`, `TreeItem`, `DataGrid`, `List`, `ListItem`, `Text`, `Document`, `Window`, `Pane`, `Table`, `Tab`

### App Compatibility
| App Type | UIA Support | Notes |
|----------|------------|-------|
| WinForms | Full | All elements accessible |
| WPF | Full | AutomationId usually set |
| Win32 | Full | Standard controls work |
| Electron (Teams, WhatsApp, Slack) | Partial | Basic elements visible, some as generic Pane |
| Web (Chrome, Edge) | Limited | Use screenshot fallback |

---

## UIA Actions Quick Reference

| Action | Purpose | Key Params |
|--------|---------|-----------|
| `ui_tree` | Get element tree | title, depth, controlTypes, filter |
| `ui_find` | Find elements | name, automationId, controlType, scope |
| `ui_invoke` | Click/toggle/select | ref, pattern (invoke/toggle/select) |
| `ui_set_value` | Type text into field | ref, value |
| `ui_get_value` | Read element value | ref |
| `ui_expand` | Expand/collapse node | ref, state (expand/collapse) |
| `ui_select` | Select list/tree item | ref |
| `ui_table` | Read grid/table data | ref, startRow, maxRows |
| `ui_clear_cache` | Clear ref cache | — |

---

## Test Workflows

### 1. Basic UI Test (No Debugging)
```
1. screen action:"process_start" path:"MyApp.exe"
2. (wait 3-5 seconds for app to load)
3. screen action:"ui_tree" title:"MyApp" depth:3
   → map out the UI: buttons, fields, tabs
4. screen action:"ui_find" title:"MyApp" name:"Username" controlType:"Edit"
   → find the input field, get ref
5. screen action:"ui_set_value" ref:"uia_5" value:"admin"
   → type username
6. screen action:"ui_find" title:"MyApp" name:"Password" controlType:"Edit"
7. screen action:"ui_set_value" ref:"uia_8" value:"secret"
   → type password
8. screen action:"ui_find" title:"MyApp" name:"Login" controlType:"Button"
9. screen action:"ui_invoke" ref:"uia_12"
   → click Login
10. (wait for response)
11. screen action:"ui_find" title:"MyApp" name:"Welcome"
    → verify success message appeared
12. screen action:"ui_get_value" ref:"uia_15"
    → read the welcome text
```

### 2. Debug + UI Test (Full E2E)
```
== SETUP ==
1. vs action:"open_file" target:"LoginController.cs" options:{line:42}
2. vs action:"breakpoint_add" target:"LoginController.cs" options:{line:42}
3. vs action:"start_debug"
   → app launches, wait for main window

== INTERACT ==
4. screen action:"ui_tree" title:"MyApp" depth:3
5. screen action:"ui_set_value" ref:"uia_5" value:"testuser"
6. screen action:"ui_invoke" ref:"uia_12"
   → clicks Submit → breakpoint hits at line 42

== INSPECT ==
7. vs_query what:"debug_state"
   → confirm Break mode
8. vs_query what:"locals"
   → see: username="testuser", password="secret"
9. vs_query what:"expression" target:"userService.IsValid(username)"
   → evaluate: true
10. vs_query what:"callstack"
    → see call chain

== STEP ==
11. vs action:"step_over"
12. vs_query what:"locals"
    → see updated variables after the line executed
13. vs action:"continue"
    → resume execution

== VERIFY ==
14. screen action:"ui_find" title:"MyApp" name:"Dashboard"
    → verify Dashboard is visible after login
15. screen action:"ui_table" ref:"uia_20"
    → read data from a table on the dashboard
```

### 3. Navigate Complex UI
```
== Tree/menu navigation ==
1. screen action:"ui_tree" title:"MyApp" controlTypes:"TreeItem" depth:5
   → find tree items
2. screen action:"ui_expand" ref:"uia_3" state:"expand"
   → expand "Settings" node
3. screen action:"ui_find" ref:"uia_3" name:"Email" controlType:"TreeItem" scope:"children"
   → find "Email" under "Settings"
4. screen action:"ui_select" ref:"uia_8"
   → select Email settings

== Tab navigation ==
5. screen action:"ui_find" title:"MyApp" controlType:"TabItem"
   → list all tabs
6. screen action:"ui_select" ref:"uia_15"
   → switch to "Advanced" tab

== Table inspection ==
7. screen action:"ui_find" title:"MyApp" controlType:"DataGrid"
8. screen action:"ui_table" ref:"uia_22" maxRows:10
   → read first 10 rows
9. screen action:"ui_table" ref:"uia_22" startRow:10 maxRows:10
   → read next 10 rows (pagination)
```

### 4. Desktop App Login Test (Generic)

```
== Example: Testing any desktop application with login ==
1. vs action:"start_no_debug"
   → launches the app
2. (wait for login dialog)
3. screen action:"ui_find" title:"Login" controlType:"Edit"
   → find username/password fields
4. screen action:"ui_set_value" ref:"uia_N" value:"testuser"
5. screen action:"ui_set_value" ref:"uia_M" value:"testpass"
6. screen action:"ui_find" title:"Login" name:"OK" controlType:"Button"
7. screen action:"ui_invoke" ref:"uia_K"
   → login
8. (wait for main window)
9. screen action:"ui_tree" title:"MainWindow" depth:3
   → explore main UI
```

## Fallback Strategy

If UIA doesn't find an element or can't interact:

1. **Try broader search**: remove controlType filter, use only name
2. **Try deeper tree**: increase depth to 5-8
3. **Fall back to screenshot**: `screenshot` → Read PNG → calculate coordinates → `mouse_click`
4. **Fall back to keyboard**: `keyboard_send` for shortcuts (Ctrl+Tab, Alt+F, etc.)

---

## Key Rules

1. **Always `ui_tree` first** — understand the UI structure before interacting
2. **Use refs** — never re-search for the same element, use its ref
3. **Check ref validity** — if element error, re-fetch with `ui_find`
4. **Prefer UIA over screenshots** — faster, more reliable, fewer tokens
5. **Use screenshots for verification** — when visual layout verification is needed
6. **Combine with VS debugging** — set breakpoints before clicking UI elements
7. **Wait after actions** — sleep 1-3s after process_start, login, etc.
8. **Record test flows in KB** — save successful test recipes for reuse

---

## Workflow / Workflow System (wf_* tools)

Structured database for storing, executing, and learning from test workflows. DB: `.roslyn-mcp/workflows.db`.

### Core Concept
Workflows are **reusable workflows** (not just tests). AI writes instructions for itself, executes them, tracks progress, and learns from results. One workflow can call another (nested execution with cycle detection).

### CRUD Tools
| Tool | Purpose |
|------|---------|
| `wf_create` | Create workflow with id, name, description, tags |
| `wf_get` | Get workflow with all steps |
| `wf_list` | List workflows, filter by tags |
| `wf_update` | Update name/description/tags |
| `wf_delete` | Delete workflow and all steps |
| `wf_clone` | Clone workflow to new ID |

### Step Tools
| Tool | Purpose |
|------|---------|
| `wf_add_step` | Add step with action, target, params, aiHint, waitTrigger |
| `wf_edit_step` | Edit step fields |
| `wf_move_step` | Reorder step |
| `wf_delete_step` | Remove step |
| `wf_enable_step` | Enable/disable step without deleting |
| `wf_learn` | Record learned notes and fallback for a step |

### Execution Tools
| Tool | Purpose |
|------|---------|
| `wf_run` | Run workflow from step N |
| `wf_run_step` | Execute single step (debug) |
| `wf_status` | Current runner state, progress, pending AI instructions |
| `wf_stop` | Stop running workflow |

### Background Watchers
| Tool | Purpose |
|------|---------|
| `wf_watch` | Start background watcher (window, element, process) |
| `wf_watch_status` | Check watcher result |
| `wf_watch_cancel` | Cancel watcher |
| `wf_watch_list` | List all active watchers |

### AI Annotations (self-instructions)
| Tool | Purpose |
|------|---------|
| `wf_annotate` | Add pre/post/observe instruction to a step |
| `wf_get_annotations` | Read all annotations for a step |

### Progress Tracking
| Tool | Purpose |
|------|---------|
| `wf_progress` | Log current step/status/message |
| `wf_get_progress` | See full progress log (current position and history) |
| `wf_clear_progress` | Clear progress log |

### History
| Tool | Purpose |
|------|---------|
| `wf_history` | Run history for a workflow |
| `wf_last_error` | Details of last failed run |

### Step Actions
- **UIA**: `uia_click`, `uia_set_value`, `uia_assert`
- **Screen**: `screen_click`, `screen_type`, `screenshot`
- **System**: `keyboard`, `command`, `process_start`, `process_kill`
- **Flow**: `call_case` (nested workflow), `wait_timeout`, `log`
- **AI**: `ai_screenshot_check`, `ai_analyze`, `ai_decide`, `ai_freeform` (pause for AI)

### Wait Triggers
Steps can wait before executing: `window:Title`, `element:Window:Name`, `process:name`, `timeout:3000`

### Waiting for User Actions
Use `wf_watch` with appropriate type to monitor user actions in background:
```
wf_watch type:"window" target:"Settings"  → wait for user to open Settings
wf_watch type:"element" target:"MainWindow:btnOK"  → wait for OK button to appear
wf_progress workflowId:"my:workflow" status:"waiting_user" message:"Waiting for user to click OK..."
```
Then poll `wf_watch_status` — when user completes their action, the watcher triggers and workflow continues.

### Self-Learning Workflow
1. **Before step**: read `aiHint` and `pre` annotations
2. **Execute step**: track with `wf_progress`
3. **After step**: check result, write `post` annotations
4. **On failure**: record via `wf_learn` (learnedNotes + fallbackAction)
5. **Next session**: read annotations and learned_notes to avoid past mistakes

### Example: Plugin Reinstall Workflow
```
wf_get id:"roslyn:reinstall-plugin"
→ 7 steps: switch_instance → stop_debug → rebuild → start_debug → wait → open_solution → verify
Each step has aiHint (what to do) and learnedNotes (what went wrong before)
```

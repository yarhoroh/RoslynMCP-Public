---
name: roslyn-debug-monitor
description: "Debug Monitor — live app state during debug: windows, dialogs, events, blocking waitFor. Zero sleeps.\nTRIGGER when: debugging an app, checking for dialogs/popups, waiting for windows, UI automation testing with debug session.\nDO NOT TRIGGER when: not debugging, only analyzing code, only VS IDE operations without running app."
---

# Debug Monitor — Instruction Reference

## Tool: `debug_monitor`

One MCP tool. Auto-activates when debug starts, auto-stops when debug ends.

### Parameters

| Param | Type | Default | Description |
|-------|------|---------|-------------|
| `events` | int | 10 | Number of recent events (1-200) |
| `waitFor` | string | null | Blocking wait condition |
| `timeout` | int | 30000 | Timeout in ms for waitFor |

### waitFor Conditions

| Condition | Waits until |
|-----------|-------------|
| `window:Title` | Window with title containing "Title" appears |
| `window_gone:Title` | Window with title disappears |
| `dialog` | Any modal dialog appears |
| `no_dialogs` | All modal dialogs gone |
| `change` | Any window create/destroy event |

---

## Response Structure

### Each window contains:

| Field | Type | Usage |
|-------|------|-------|
| `handle` | int | **Use directly in `ui_find handle:`** — no conversion needed |
| `hwnd` | hex string | For matching with events log only |
| `title` | string | Window title |
| `type` | "Window" / "Dialog" | Window type |
| `focused` | bool | Is foreground window |
| `modal` | bool | Is modal dialog |
| `elements` | array/null | All actionable UI elements on the form |

### Each element contains:

| Field | Present in | Usage |
|-------|-----------|-------|
| `type` | all | button, input, text, combobox, checkbox, listitem, tab |
| `name` | all except text | Element name |
| `automationId` | all except text | **Use in `ui_click automationId:`** — most reliable identifier |
| `value` | input, text | Current value / displayed text |
| `bounds` | all | `{x, y, w, h}` — screen coordinates, for direct mouse_click if needed |
| `isChecked` | checkbox | Checked state |

---

## Action: `screen ui_click` — Find + Invoke in One Call

Use `ui_find` to locate elements, then `ui_invoke` to interact:

```json
// 1. Find element:
screen { "action": "ui_find", "handle": 12345, "automationId": "btnOK" }
// → ref: "uia_5"

// 2a. Single click (button, menu item):
screen { "action": "ui_invoke", "ref": "uia_5" }
// Uses InvokePattern. Falls back to click center if unsupported.

// 2b. Double click (open ListItem, tree node — WinForms):
screen { "action": "ui_invoke", "ref": "uia_5", "pattern": "double_click" }
// Double-clicks center of element bounds. No coordinate math needed.

// 2c. Select item in list/tree:
screen { "action": "ui_invoke", "ref": "uia_5", "pattern": "select" }
// Uses SelectionItemPattern.

// 2d. Toggle checkbox:
screen { "action": "ui_invoke", "ref": "uia_5", "pattern": "toggle" }
```

### Available patterns for ui_invoke

| Pattern | What it does | When to use |
|---------|-------------|-------------|
| `invoke` (default) | InvokePattern → click button | Buttons, menu items |
| `double_click` | Double-click center of bounds | Open ListItem, TreeItem in WinForms |
| `toggle` | TogglePattern → check/uncheck | Checkboxes |
| `select` | SelectionItemPattern → select | List/tree selection without opening |

## App Launch Monitoring

Three commands cover everything:

| Command | What it does |
|---------|-------------|
| `debug_monitor events:N` | Instant snapshot — what windows exist right now |
| `debug_monitor waitFor:"change"` | Block until ANY window event (created/destroyed) |
| `debug_monitor waitFor:"window:text"` | Block until window with title containing "text" appears |

### Universal launch workflow

```
1. vs start_debug
2. debug_monitor events:3               ← snapshot: what's there?
   → windows: []                       ← nothing yet, app loading
3. debug_monitor events:3               ← poll again
   → windows: ["SplashForm"]           ← splash appeared
4. debug_monitor waitFor:"change"       ← wait for next event (splash gone, main window)
   → read windows[].title              ← see what appeared
5. Repeat until main window is there
   → read windows[].elements[]         ← app ready, all UI elements available
```

No hardcoded window names. Just observe → wait → observe.

## Tool Priority for App Interaction

Always use tools in this order — higher priority first:

| Priority | Tool | When |
|----------|------|------|
| 1 (best) | `ui_find` → `ui_invoke` / `ui_select` | Default for ALL UI interaction |
| 2 | `mouse_click` | Fallback when UIA blocks (breakpoint freeze, modal, InputBox) |
| 3 | `screenshot` | Last resort — verify visual state when tools give no info |

## UI Interaction During Debug — Decision Tree

When breakpoints are active, `ui_invoke`/`ui_select` may block (UIA waits for app response, but app is frozen by debugger → 100s timeout). Follow this algorithm:

```
Click needed during debug?
       │
       ▼
Try ui_find → ui_invoke/ui_select first (Priority 1)
       │
       ├── Succeeded → done, check vs_query debug_state
       │
       └── Timed out (100s) → app frozen (breakpoint or modal)
              │
              ▼
       vs_query what:"debug_state"
              │
              ├── mode:"Break" → breakpoint hit! → inspect locals/callstack → step/continue
              │
              └── mode:"Run" → not a breakpoint, something else blocked
                     │
                     ▼
              Fallback: mouse_click by coords (Priority 2)
                     │
                     └── Still stuck? → screenshot (Priority 3) → check for dialog
```

**Key insight:** If you KNOW a breakpoint will fire (you just set it), skip straight to `mouse_click` to avoid the 100s wait. But if unsure — always try `ui_invoke` first.

### Situations & Actions

| Situation | Action |
|-----------|--------|
| No breakpoints, normal interaction | `ui_find` → `ui_invoke` / `ui_select` |
| Breakpoint set, you KNOW click will trigger it | `ui_find` (get coords) → `mouse_click` → `vs_query debug_state` |
| Breakpoint set, click may NOT trigger it | Try `ui_invoke` first. If timeout → `mouse_click` fallback |
| `ui_invoke` timed out (100s) | Check `vs_query debug_state`. If Break → inspect & continue. If Run → `mouse_click` fallback, then `screenshot` |
| After `continue` — verify app resumed | `vs_query debug_state` → confirm mode:"Run". If still Break → another breakpoint, inspect again |
| Need to verify app visual state | `screenshot` the app window |

### Correct Breakpoint Debug Flow (tested & verified)

```
1. vs breakpoint_add target:"file.cs" options:{line: N}
2. vs start_debug
3. screen window_activate processName:"MyApp"
4. screen ui_find → get element ref and bounds
5. screen ui_invoke ref:"uia_N"             ← try UIA first (Priority 1)
   └── if timeout: mouse_click x:... y:...  ← fallback (Priority 2)
6. vs_query what:"debug_state"              ← immediately check
7. vs_query what:"locals"                   ← inspect variables
8. vs_query what:"callstack"                ← see call chain
9. vs step_over                             ← step through code
10. vs_query what:"locals"                  ← see updated values
11. vs breakpoint_clear_all                 ← clean up
12. vs continue                             ← resume app
13. vs_query what:"debug_state"             ← confirm mode:"Run"
```

## Rules

1. **waitFor replaces ALL sleeps** — WinEventHook fires within milliseconds
2. **vs stepping is sync-by-default (v1.18.6+)** — `step_over`, `step_into`, `step_out`, `continue` all WAIT and return debugState. Use debug_monitor only for window/dialog events, NOT for waiting after step commands.
3. **handle + ui_find + ui_invoke** — use handle from monitor, find element, invoke it
4. **automationId is preferred** for ui_find; fall back to name + controlType when null
5. **bounds enable direct mouse_click** — for double-click or elements where ui_invoke doesn't work
6. **Ignore TimerNativeWindow** events — WinForms timer noise
7. **Always check `conditionMet`** in waitResult — false means timeout
8. **Auto-start fallback** — if debug is running but monitor inactive, calling debug_monitor will auto-start it

## Lifecycle

1. `vs start_debug` → monitor auto-starts for debugged process
2. **Stepping:** Use `vs step_over/step_into/step_out/continue` (sync-by-default) — they return debugState automatically. Do NOT use debug_monitor for step synchronization.
3. **Windows/dialogs:** Use `debug_monitor` with `waitFor` — instant snapshot or blocking wait for window events
4. Debug stops → monitor auto-stops
5. Inactive monitor returns `{ active: false }`

## Dashboard

Debug Monitor tab in VS RoslynMCP Dashboard shows same data: windows, events, status.

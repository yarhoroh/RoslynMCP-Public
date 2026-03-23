---
name: roslyn-screen
description: "Full computer automation — screenshots, mouse, keyboard, window/process management, clipboard, UI Automation.\nTRIGGER when: ANY desktop automation — screenshots, mouse clicks, keyboard input, window/process management, clipboard.\nDO NOT TRIGGER when: only working with code/files without desktop interaction."
---

# Desktop Automation — Complete Reference

> MCP tool `screen` with `action` parameter. 23 actions for full desktop control via Win32 API + UI Automation.

**Save workflows as workflows!** When you discover a working automation sequence, save it:
```
wf_create id:"app:task" name:"Task Name" tags:"automation"
wf_add_step workflowId:"app:task" action:"ai_freeform" aiHint:"Detailed steps..."
```
Example: `wf_get id:"teams:send-message"` — 7-step AI-driven Teams messaging workflow with learned notes.

---

## Knowledge Persistence Protocol

After completing a successful automation workflow:
1. **Record the recipe** in KB via `kb_add` (type: Snippet, category: tech)
2. **Include**: target app, steps taken, coordinates/parameters that worked, gotchas
3. **Tag** with app name and action type (e.g., `teams`, `messaging`, `self-update`)
4. Before starting a new workflow, **search KB first**: `kb_search query:"Teams message"`
5. Record **lessons learned** (failures, workarounds) via `memory_remember` (Lesson, global, importance >= 7)
6. **Don't record**: bug fixes (they're in git), session-specific context (use session memory)

---

## Actions Reference

### screenshot — Capture screen, window, or region
```json
screen { "action": "screenshot" }
screen { "action": "screenshot", "target": "window", "title": "Teams", "scale": 0.4 }
screen { "action": "screenshot", "target": "region", "x": 100, "y": 100, "width": 800, "height": 600, "scale": 1.0 }
```
| Parameter | Values | Default | Notes |
|-----------|--------|---------|-------|
| target | screen, window, region | screen | What to capture |
| scale | 0.1–1.0 | 0.5 | Lower = smaller image, fewer tokens |
| monitor | 0, 1, 2... | 0 | For target=screen only |
| title | string | — | Window title substring match |
| handle | number | — | Window handle (from window_list) |
| x, y, width, height | numbers | — | For target=region only |

Returns: `{ filePath, width, height, originalWidth, originalHeight, scale, fileSizeKB }`

**View** the screenshot with the `Read` tool (multimodal image support).

#### Coordinate Mapping (Critical!)
```
Full-screen:  real_X = image_X / scale
Window:       real_X = window_X + (image_X / scale)
              real_Y = window_Y + (image_Y / scale)
```
- `window_X/Y` comes from `window_list` or `window_activate` response
- Example: image coord (400, 300) at scale=0.4, window at (1791, 166)
  → real screen = (1791 + 400/0.4, 166 + 300/0.4) = (2791, 916)

### mouse_click — Click at screen coordinates
```json
screen { "action": "mouse_click", "x": 2791, "y": 916 }
screen { "action": "mouse_click", "x": 500, "y": 300, "button": "right" }
screen { "action": "mouse_click", "x": 500, "y": 300, "clickType": "double" }
```
| Parameter | Values | Default |
|-----------|--------|---------|
| x, y | screen coordinates | required |
| button | left, right, middle | left |
| clickType | single, double | single |

### mouse_move — Move cursor without clicking
```json
screen { "action": "mouse_move", "x": 500, "y": 300 }
```

### mouse_scroll — Scroll wheel
```json
screen { "action": "mouse_scroll", "x": 500, "y": 300, "delta": -3 }
screen { "action": "mouse_scroll", "delta": 5, "direction": "horizontal" }
```
| Parameter | Values | Default | Notes |
|-----------|--------|---------|-------|
| delta | integer | -3 | Negative = down/left, positive = up/right |
| direction | vertical, horizontal | vertical | |

### keyboard_send — Type text and key combos
```json
screen { "action": "keyboard_send", "input": "Hello World{Enter}" }
screen { "action": "keyboard_send", "input": "{Ctrl+A}{Delete}" }
screen { "action": "keyboard_send", "input": "Привет!{Enter}", "delay": 15 }
```

**Special keys**: `{Enter}`, `{Tab}`, `{Escape}`, `{Backspace}`, `{Delete}`, `{Space}`, `{Up}`, `{Down}`, `{Left}`, `{Right}`, `{Home}`, `{End}`, `{PageUp}`, `{PageDown}`, `{F1}`–`{F12}`, `{Win}`

**Combos**: `{Ctrl+S}`, `{Alt+F4}`, `{Ctrl+Shift+P}`, `{Ctrl+A}`, `{Ctrl+V}`, `{Ctrl+C}`, `{Ctrl+Z}`

**Special keys (new)**: `{Insert}`, `{Ins}`, `{PrintScreen}`, `{PrtSc}`, `{Pause}`, `{NumLock}`, `{ScrollLock}`, `{CapsLock}`, `{Apps}`

**Unicode**: Cyrillic, CJK, emoji are automatically pasted via clipboard (set clipboard on VS UI thread, then Ctrl+V from background thread to not steal focus from target app). Direct `KEYEVENTF_UNICODE` does not work in Electron/Chromium apps (Teams, Slack, VS Code, Chrome).

### window_list — Enumerate all visible windows
```json
screen { "action": "window_list" }
```
Returns: `{ windows: [{ title, handle, processName, pid, x, y, width, height, isMinimized }], count }`

### window_activate — Bring window to foreground
```json
screen { "action": "window_activate", "title": "Teams" }
screen { "action": "window_activate", "handle": 12345 }
screen { "action": "window_activate", "processName": "ms-teams" }
```
Automatically restores minimized windows. Title uses substring match (case-insensitive).

### window_resize — Move, resize, minimize, maximize
```json
screen { "action": "window_resize", "title": "Notepad", "x": 0, "y": 0, "width": 1920, "height": 1080 }
screen { "action": "window_resize", "title": "Teams", "windowAction": "maximize" }
```
`windowAction`: minimize, maximize, restore (overrides x/y/w/h)

### process_list — List running processes
```json
screen { "action": "process_list" }
screen { "action": "process_list", "filter": "chrome", "withWindow": "true" }
```

### process_start — Launch application
```json
screen { "action": "process_start", "path": "notepad.exe" }
screen { "action": "process_start", "path": "C:\\app.exe", "arguments": "--flag", "workingDirectory": "C:\\dir" }
```

### process_kill — Terminate process
```json
screen { "action": "process_kill", "pid": 12345 }
screen { "action": "process_kill", "name": "notepad" }
```

### clipboard_get / clipboard_set — Clipboard access
```json
screen { "action": "clipboard_get" }
screen { "action": "clipboard_set", "text": "Hello World" }
```

### screen_info — Monitor and cursor information
```json
screen { "action": "screen_info" }
```
Returns: monitors (device, resolution, DPI, scaleFactor, workArea), cursor position.

---

## UI Automation (UIA) — Programmatic UI Access

> 9 actions for direct element access via Windows UI Automation. No screenshots, no coordinates, no mouse.
> Works with: WinForms, WPF, Win32 (full support), Electron/web apps (partial).

### ui_tree — Get element tree of a window
```json
screen { "action": "ui_tree", "title": "MyApp" }
screen { "action": "ui_tree", "title": "MyApp", "depth": 5, "controlTypes": "Button,Edit,TabItem" }
screen { "action": "ui_tree", "ref": "uia_42", "depth": 2 }
```
| Parameter | Values | Default | Notes |
|-----------|--------|---------|-------|
| title / handle / processName | string/int | foreground window | Target window |
| ref | string | — | Start from cached element instead of window root |
| depth | 1–10 | 3 | Max tree depth |
| maxElements | integer | 200 | Limit output size |
| filter | string | — | Substring match on Name or AutomationId |
| controlTypes | string | — | Comma-separated: Button, Edit, ComboBox, TabItem, MenuItem, etc. |

Returns: `{ tree: { ref, name, controlType, children[] }, totalElements, truncated }`

**Every returned element gets a `ref` (e.g., `uia_42`) — use it in subsequent calls.**

### ui_find — Find elements by properties
```json
screen { "action": "ui_find", "title": "MyApp", "name": "Submit", "controlType": "Button" }
screen { "action": "ui_find", "ref": "uia_5", "controlType": "Edit", "scope": "children" }
screen { "action": "ui_find", "title": "MyApp", "automationId": "txtUsername" }
```
| Parameter | Values | Default | Notes |
|-----------|--------|---------|-------|
| name | string | — | Substring match on element Name |
| automationId | string | — | Exact match on AutomationId |
| controlType | string | — | Single type: Button, Edit, MenuItem, etc. |
| className | string | — | WinForms/WPF class name (exact) |
| scope | children, descendants | descendants | Search scope |
| maxResults | integer | 20 | Limit results |

### ui_invoke — Click button, toggle checkbox, select item

```json
screen { "action": "ui_invoke", "ref": "uia_5" }
screen { "action": "ui_invoke", "ref": "uia_12", "pattern": "toggle" }
screen { "action": "ui_invoke", "ref": "uia_8", "pattern": "select" }
screen { "action": "ui_invoke", "ref": "uia_8", "pattern": "double_click" }
```
| pattern | What it does | When to use |
|---------|-------------|-------------|
| invoke (default) | InvokePattern — click button/menu | Buttons, menu items. Falls back to click center |
| toggle | TogglePattern — checkbox on/off | Checkboxes |
| select | SelectionItemPattern — select in list | Select without opening |
| double_click | Double-click center of element bounds | Open ListItem/TreeItem in WinForms |

### ui_set_value — Type text into a field
```json
screen { "action": "ui_set_value", "ref": "uia_8", "value": "Hello World" }
```
Uses ValuePattern directly. Falls back to focus + Ctrl+A + type if ValuePattern unavailable.

### ui_get_value — Read element state
```json
screen { "action": "ui_get_value", "ref": "uia_8" }
```
Returns: `{ ref, name, controlType, value, text, toggleState, isSelected, rangeValue }`
Reads all applicable patterns: Value, Text, Toggle, Selection, RangeValue.

### ui_expand — Expand/collapse tree nodes, combos
```json
screen { "action": "ui_expand", "ref": "uia_15", "state": "expand" }
screen { "action": "ui_expand", "ref": "uia_15", "state": "collapse" }
```

### ui_select — Select item in list/tree/tab
```json
screen { "action": "ui_select", "ref": "uia_22" }
```
Uses SelectionItemPattern. Falls back to InvokePattern, then click.

### ui_table — Read DataGrid/Table data
```json
screen { "action": "ui_table", "ref": "uia_10" }
screen { "action": "ui_table", "ref": "uia_10", "startRow": 50, "maxRows": 25 }
```
Returns: `{ rowCount, columnCount, headers[], rows[{ index, cells[] }], truncated }`
Supports pagination via startRow/maxRows.

### ui_clear_cache — Clear element reference cache
```json
screen { "action": "ui_clear_cache" }
```
Element refs expire after 120 seconds. Call this to force-clear.

---

## Workflows

### Test Application with UIA (Preferred for native apps)
```
1. process_start path:"app.exe"
2. ui_tree title:"MyApp" depth:3                       → get element tree
3. ui_find title:"MyApp" name:"Username" controlType:"Edit"  → find input
4. ui_set_value ref:"uia_8" value:"testuser"           → type directly
5. ui_find title:"MyApp" name:"Submit" controlType:"Button"  → find button
6. ui_invoke ref:"uia_12"                              → click programmatically
7. ui_find title:"MyApp" name:"Error" controlType:"Text"     → check result
8. ui_get_value ref:"uia_15"                           → read error message
9. Record working flow in KB
```

### Full Debug + UIA Testing (with VS)
```
1. vs action:"breakpoint_add" target:"Login.cs" options:{line:42}
2. vs action:"start_debug"                             → launch app
3. ui_tree title:"MyApp" depth:3                       → get UI elements
4. ui_set_value ref:"uia_5" value:"test"               → fill form
5. ui_invoke ref:"uia_8"                               → click button → hits breakpoint
6. vs_query what:"locals"                              → inspect variables
7. vs action:"step_over"                               → step through code
8. vs action:"continue"                                → resume
9. ui_find title:"MyApp" name:"Success"                → verify UI result
```

### When to use UIA vs Screenshots
| Scenario | Use |
|----------|-----|
| Native app (WinForms, WPF, Win32) | **UIA** — full element access |
| Read text/data from app | **UIA** — direct, no OCR |
| Click buttons, fill forms | **UIA** — no coordinate math |
| Electron app (Teams, Slack, WhatsApp) | **UIA first**, screenshot fallback |
| Visual verification (layout, colors) | **Screenshot** — UIA can't see pixels |
| App with poor accessibility | **Screenshot + mouse** — last resort |

### Send a Message in a Chat App
```
1. window_activate title:"Teams"                     → activate window, note x/y
2. screenshot target:window title:"Teams" scale:0.4  → see current state
3. Read the PNG file                                 → analyze UI, find input field
4. Calculate click coords: image_coord / scale + window_offset
5. mouse_click x:X y:Y                              → click input field
6. keyboard_send input:"Message text{Enter}"         → type and send
7. screenshot                                        → verify message sent
8. kb_add the recipe as Snippet                      → persist for reuse
```

### Test UI of an Application
```
1. process_start path:"app.exe"
2. screenshot → analyze UI layout
3. mouse_click / keyboard_send → interact with elements
4. screenshot → verify result
5. Repeat as needed
6. Record working flow in KB
```

### Self-Update RoslynMCP Extension

```
⚠️ CRITICAL: Roslyn MCP tools DON'T WORK when VS is closed!
⚠️ NEVER kill devenv without asking user — they may have unsaved work.
⚠️ Exit code 212 from VSIXInstaller = VS is still running, close it first.

CORRECT ORDER:
1. MSBuild the .csproj (//t:Build //p:Configuration=Debug)
2. Ask user to close VS (or confirm it's safe to close)
3. taskkill //IM MSBuild.exe //F (prevent exit code 212)
4. VSIXInstaller //quiet //force "<path/to/Extension.vsix>"
5. Verify exit code = 0
6. Start devenv.exe "<path/to/solution.sln>" in background
7. Wait ~25-40s for VS to load
8. Verify Roslyn MCP tools respond (search_tools or memory_context)
```

### Navigate a Complex UI (Scrolling)
```
1. screenshot → see visible area
2. If target not visible: mouse_scroll delta:-5 → scroll down
3. screenshot → check new area
4. Repeat until target found
5. mouse_click on target
```

---

## Key Rules

1. **Always screenshot after interaction** — verify before proceeding
2. **Coordinate math is critical** — wrong coords = wrong click target
3. **Non-ASCII auto-pastes via clipboard** — no special handling needed by caller
4. **Window title is substring match** — "Teams" matches "Chat | Teams | ..."
5. **Scale tradeoff**: 0.3–0.4 for overview, 0.8–1.0 for reading text precisely
6. **Search KB before starting** — a recipe may already exist
7. **Record successful flows in KB** — save time in future sessions

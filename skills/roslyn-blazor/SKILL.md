---
name: roslyn-blazor
description: "BlazorPilot — Blazor/Web UI automation via CDP + Roslyn. Inspect .razor components, control browser UI (click, type, select, screenshot), read DevTools (console, network, cookies). Works with Blazor Desktop (WebView2), Blazor Server/WASM (browser), and any website."
user_invocable: true
trigger: "TRIGGER when: ANY Blazor UI interaction, web automation, browser control, testing Blazor app UI, inspecting .razor components, reading console/network logs, taking screenshots of web apps.\nDO NOT TRIGGER when: only analyzing C# code without UI interaction (use roslyn-code), or only VS IDE commands (use roslyn-vs)."
---

# BlazorPilot — Blazor & Web UI Automation

> **One tool `blazor` with 35 actions.** Works with Blazor Desktop, Blazor Server/WASM, and ANY website in Chrome/Edge.

## How It Works

```
Roslyn (static)  →  Parses .razor files: components, events, bindings, state → C# handler file:line
CDP (runtime)    →  Connects to browser/WebView2: accessibility tree, clicks, input, DevTools
Combined         →  AI sees HTML element + knows which C# method handles it
```

---

## Connection Setup

### Blazor Desktop (WebView2) — e.g. MAUI, WPF with BlazorWebView

WebView2 needs environment variable to enable CDP port. `blazor connect` auto-adds it to launchSettings.json.

**Automatic (recommended):**
```json
blazor action:"connect"
// 1. Searches launchSettings.json in solution
// 2. Adds WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS if missing
// 3. Connects to CDP port 9222
// If app was running without the env var → restart app (Ctrl+Shift+F5)
```

**Manual (if auto fails):**
Add to `Properties/launchSettings.json` in the Desktop project:
```json
"environmentVariables": {
    "WEBVIEW2_ADDITIONAL_BROWSER_ARGUMENTS": "--remote-debugging-port=9222"
}
```
Then restart the app.

### Blazor Server / WASM (browser) — Web Application

App runs in normal browser. Need to launch browser with CDP port:
```bash
# Edge
msedge --remote-debugging-port=9222 --user-data-dir=<temp> http://localhost:5258

# Chrome
chrome --remote-debugging-port=9222 --user-data-dir=<temp> http://localhost:5258
```
App URL from `launchSettings.json` → `applicationUrl` field.

Then connect:
```json
blazor action:"connect" port:9222
```

### Any Website (Chrome/Edge)

Launch browser with CDP port and navigate to any URL:
```bash
chrome --remote-debugging-port=9223 --no-first-run --user-data-dir=<temp> https://github.com
```
**Note:** `--user-data-dir=<temp>` creates a clean profile (no logins/cookies). This is because Chrome doesn't allow two instances with the same profile. The user's main Chrome is unaffected.

To use with user's profile: close all Chrome windows first, then launch with `--remote-debugging-port` and the real user-data-dir path.

```json
blazor action:"connect" port:9223
```

---

## Static Analysis (Roslyn) — No browser needed

### inspect — Full .razor page analysis
```json
blazor action:"inspect" filePath:"Shared/Shared.UI/Components/Pages/Translator.razor"
```
Returns:
- **routes**: `["/", "/translator"]`
- **components**: name, line, parameters, events with handler location (file:line)
- **elements**: HTML elements with @onclick, @bind, @onchange
- **state**: all properties/fields with types, defaults, [Parameter]/[Inject] flags
- **methods**: all handlers with signatures, async flag, file:line
- **actions**: ready-to-use commands (click selector, type selector, etc.)

### list_pages — All Blazor pages in solution
```json
blazor action:"list_pages"
```

---

## Page Observation (CDP)

### accessibility_tree — Enhanced with HTML snippets
```json
blazor action:"accessibility_tree"
```
Returns text tree of all visible elements. **Interactive elements include HTML snippets:**
```
[button] "Swap languages"
  html: <button class="inline-flex items-center..." title="Swap languages">
[textbox] "Start speaking or type here..."
  html: <textarea class="flex-1 w-full px-4..." placeholder="Start speaking or type here...">
[combobox] value="Microphone (Razer Seiren)"
  html: <select class="flex-1 text-sm..."><option value="...">...
```
AI sees role, name, value AND the actual HTML — knows exactly how to interact.

### snapshot — Compact page summary
```json
blazor action:"snapshot"
```
Returns: url, title, all named elements with role/name/value.

### screenshot — Full page or element
```json
blazor action:"screenshot"                              // full page
blazor action:"screenshot" selector:"textarea"          // specific element
```
Saves PNG to temp file. Use Read tool to view.

---

## UI Actions (CDP)

All actions use **CDP Input.dispatchMouseEvent** — real browser clicks that work with Blazor EventCallbacks.

### click — By CSS selector
```json
blazor action:"click" selector:"button[title='Swap languages']"
blazor action:"click" selector:".flex-1.min-w-0 button"
blazor action:"click" selector:"#myButton"
```
Returns: HTML snippet of clicked element.

### click_by_text — By visible text (smart)
```json
blazor action:"click_by_text" value:"English"
blazor action:"click_by_text" value:"Settings"
blazor action:"click_by_text" value:"Save"
```
**Priority order:**
1. Buttons inside overlay/modal (`.fixed.inset-0`, `[role=dialog]`)
2. All `<button>` elements on page
3. `<a>`, `[role=button]` elements
4. Any clickable element

Also searches by `title` and `aria-label` attributes.
Auto-scrolls element into view before clicking.

### type — Enter text
```json
blazor action:"type" selector:"textarea" value:"Hello World"
blazor action:"type" selector:"input[placeholder='Search...']" value:"query"
blazor action:"type" selector:"input[type='email']" value:"test@example.com"
```

### press_key — Keyboard key
```json
blazor action:"press_key" value:"Enter"
blazor action:"press_key" value:"Tab"
blazor action:"press_key" value:"Escape"
blazor action:"press_key" value:"ArrowDown"
```

### select — By value or text (auto-detects)
```json
blazor action:"select" selector:"select" value:"Option text here"
blazor action:"select" selector:"select.my-class" value:"option-value"
```
Tries value match first, then text match. Returns selected option text.

### select_by_text — By visible option text
```json
blazor action:"select_by_text" selector:"select" value:"Line 1 (Virtual Audio Cable)"
```

### clear — Clear input/textarea
```json
blazor action:"clear" selector:"textarea"
blazor action:"clear" selector:"input[name='search']"
```

### focus — Focus element
```json
blazor action:"focus" selector:"textarea"
```

### hover — Mouse hover
```json
blazor action:"hover" selector:"button[title='Settings']"
```

### scroll — Page or element
```json
blazor action:"scroll" value:"down"                     // page scroll down
blazor action:"scroll" value:"up"                       // page scroll up
blazor action:"scroll" selector:".list-container" value:"down"  // element scroll
```

### check / uncheck — Checkbox
```json
blazor action:"check" selector:"input[type='checkbox']"
blazor action:"uncheck" selector:"#rememberMe"
```

### wait — Wait for element
```json
blazor action:"wait" selector:".loading-complete" value:"5000"  // wait up to 5s
blazor action:"wait" selector:"#result"                         // default 5s timeout
```

### get_html — Get element HTML + attributes
```json
blazor action:"get_html" selector:".flex-1.min-w-0 button"
```
Returns: tag, outerHTML (500 chars), text content, all attributes.

---

## Navigation

### navigate — Go to URL
```json
blazor action:"navigate" url:"https://localhost:5258/settings"
```

### eval — Execute JavaScript
```json
blazor action:"eval" expression:"document.title"
blazor action:"eval" expression:"window.location.href"
blazor action:"eval" expression:"document.querySelectorAll('button').length"
```

---

## DevTools

### console — Browser console logs
```json
blazor action:"console"                    // last 50 logs
blazor action:"console" value:"5"          // last 5 logs
blazor action:"console" value:"all"        // all logs
blazor action:"console" value:"clear"      // clear buffer
```
Captures console.log, console.warn, console.error from the running app.

### network — HTTP requests with status
```json
blazor action:"network"                    // last 20 requests
blazor action:"network" value:"all"        // all requests
blazor action:"network" value:"clear"      // clear buffer
```
Returns: method, URL, status (200 OK / 404 Not Found / FAILED), error text, timestamp.

### cookies — Browser cookies
```json
blazor action:"cookies"
```

### storage — localStorage / sessionStorage
```json
blazor action:"storage" value:"local"      // localStorage (default)
blazor action:"storage" value:"session"    // sessionStorage
```

---

## CSS & Inspection

### css — Get/set computed styles
```json
blazor action:"css" selector:"textarea"                    // get styles
blazor action:"css" selector:"#myDiv" value:"color:red"    // set style
```
Get returns: display, position, width, height, color, background, font, border, opacity, etc.

### element_state — Full element state
```json
blazor action:"element_state" selector:"textarea"
```
Returns: exists, visible, enabled, checked, value, text, tag, bounding rect, all attributes.

### highlight — Visual highlight (red overlay)
```json
blazor action:"highlight" selector:"button[title='Save']"
```
Shows red overlay on element for 2 seconds. Useful for debugging which element is targeted.

### count — Count matching elements
```json
blazor action:"count" selector:"button"
blazor action:"count" selector:".error-message"
```

---

## Emulation

### viewport — Responsive testing
```json
blazor action:"viewport" value:"375x812 mobile"    // iPhone-like
blazor action:"viewport" value:"1024x768"           // tablet
blazor action:"viewport" value:"clear"              // reset to default
```

### emulate — Color scheme
```json
blazor action:"emulate" value:"dark"       // dark mode
blazor action:"emulate" value:"light"      // light mode
blazor action:"emulate" value:"clear"      // reset
```

---

## Performance

### performance — Browser metrics
```json
blazor action:"performance"
```
Returns: DOM nodes, JS event listeners, JS heap size, layout/recalc counts, DomContentLoaded time, etc.

---

## Connection Management

### connect — Connect to browser CDP
```json
blazor action:"connect"                    // auto-discover port 9222
blazor action:"connect" port:9223          // specific port
```

### disconnect — Disconnect from browser
```json
blazor action:"disconnect"
```

---

## Workflow Examples

### Test Blazor form
```
1. blazor action:"inspect" filePath:"Pages/OrderForm.razor"     → see all fields, events, handlers
2. blazor action:"connect"                                       → connect to running app
3. blazor action:"accessibility_tree"                            → see current UI state
4. blazor action:"type" selector:"input[placeholder='Name']" value:"John"
5. blazor action:"select" selector:"select" value:"Electronics"
6. blazor action:"click" selector:"button[type='submit']"
7. blazor action:"accessibility_tree"                            → verify result
8. blazor action:"network"                                       → check API calls
9. blazor action:"console"                                       → check for errors
```

### Debug UI issue
```
1. blazor action:"element_state" selector:"#submitBtn"           → check if visible/enabled
2. blazor action:"css" selector:"#submitBtn"                     → check styles
3. blazor action:"get_html" selector:"#submitBtn"                → see full HTML
4. blazor action:"highlight" selector:"#submitBtn"               → visually confirm element
5. blazor action:"screenshot"                                    → capture current state
6. blazor action:"console"                                       → check JS errors
```

### Responsive testing
```
1. blazor action:"viewport" value:"375x812 mobile"
2. blazor action:"screenshot"                                    → mobile view
3. blazor action:"viewport" value:"1024x768"
4. blazor action:"screenshot"                                    → tablet view
5. blazor action:"viewport" value:"clear"
```

---

## Key Technical Details

- **CDP clicks** use `Input.dispatchMouseEvent` (not JS `.click()`) — works with Blazor EventCallbacks
- **click_by_text** searches overlays/modals first, then page — handles Blazor popups correctly
- **accessibility_tree** filters out InlineTextBox duplicates, includes HTML for interactive elements
- **select** tries value match first, then text match — handles Blazor selects with GUID values
- **Console/Network** events captured in background via CDP event subscription
- **Auto-CDP for WebView2**: `EnsureCdpPortInLaunchSettings()` adds env var to launchSettings.json
- **Roslyn inspect** works without browser — parses .razor + .razor.cs files directly

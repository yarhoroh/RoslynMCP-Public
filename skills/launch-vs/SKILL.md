---
name: launch-vs
description: "Launch Visual Studio with a project/solution. Supports known projects, arbitrary .sln paths, or empty VS.\nTRIGGER when: user says 'launch VS', 'open solution', 'запусти проект', 'открой солюшен', or needs to start/restart Visual Studio.\nDO NOT TRIGGER when: VS is already running and user just wants to build/debug (use roslyn-vs)."
---

# Launch Visual Studio with Project

## Prerequisites

- Detect VS installation via `vswhere.exe` or check common paths
- Prefer VS Preview/Insiders over stable if available
- The proxy path in `.mcp.json` is auto-configured — do NOT change it manually

## Known Projects

Look up solutions dynamically:
```bash
# Find .sln files in current working directory
find "$(pwd)" -maxdepth 3 -name "*.sln" 2>/dev/null
```
If user mentions a project by name, search for matching .sln files.

## Workflow

### 1. Check if VS is already running
```bash
tasklist | grep -i devenv
```
If running, ask user whether to close it or open a second instance.

### 2. Resolve project
- If user gives a path → use it directly
- If user says "пустой"/"empty"/"новый" → launch devenv without arguments
- If user says a project name → search for matching .sln files:
  ```bash
  find "$(pwd)" -maxdepth 4 -name "*.sln" 2>/dev/null
  ```

### 3. Launch VS

```bash
# Find VS installation path first
VS_PATH=$(vswhere -latest -property installationPath 2>/dev/null || echo "")
MSYS_NO_PATHCONV=1 "$VS_PATH/Common7/IDE/devenv.exe" "PATH_TO_SOLUTION" &
```

### 4. Wait for VS to load (~35-45 seconds)
```bash
sleep 40 && tasklist | grep -i devenv
```

### 5. Verify MCP connection
```
mcp__roslyn__screen action: screen_info
```
If 500 error — wait 15 more seconds and retry. Max 3 retries.

### 6. Confirm to user
Report that VS is running and MCP is connected.

## VSIX Development Workflow (IMPORTANT — always follow this order!)

### Step A: Test in Experimental Instance FIRST

1. `vs action:"save_all"` → `vs action:"build"` → `vs_query what:"errors"` — check no errors
2. `vs action:"enable_deploy" target:"<YourExtensionProject>"` (once per config)
3. `vs action:"start_debug"` — launches Experimental Instance
4. Wait ~20s, then `list_instances` → `switch_instance` to Experimental Instance
5. Test new functionality there
6. `switch_instance` back to main instance
7. `vs action:"stop_debug"` — close Experimental Instance
8. If bugs found → fix → goto step 1

### Step B: Deploy to Production VS (only after Step A passes!)

1. Close all VS instances (ask user first!)
2. `taskkill //IM MSBuild.exe //F` and `taskkill //IM copilot-language-server.exe //F`
3. Build: `MSYS_NO_PATHCONV=1 "<MSBuildPath>" "<Project.csproj>" /t:Build /p:Configuration=Debug /v:quiet /nologo`
4. Install: `MSYS_NO_PATHCONV=1 "<VSIXInstallerPath>" /quiet /force "<path.vsix>"`
   - NEVER use `/u:` syntax — only `/uninstall:ID` or `/quiet /force`
5. Start VS with solution
6. Verify MCP connection

## Notes

- Never change .mcp.json proxy path manually — it's auto-configured
- Never kill devenv via MCP screen tools — MCP server runs inside VS, will deadlock
- copilot-language-server.exe is the #1 hidden blocker for VSIX install
- Always use foreground bash commands — clean up immediately if background tasks hang
- NEVER leave zombie background bash processes running


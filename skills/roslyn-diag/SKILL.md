---
name: roslyn-diag
description: "Memory/heap diagnostics via Roslyn MCP `diag` tool — ClrMD-based leak hunting for live .NET processes. 22 actions: snapshot_create (async), type_stats, find_roots, compare_snapshots, event_subscriptions, live_forms, gc_stats and more.\nTRIGGER when: user mentions memory leak, heap growing, dispose issue, unsubscribed events, retention path, slow app, memory pressure, or needs to snapshot/diff a .NET process.\nDO NOT TRIGGER when: analyzing non-.NET processes or static code review only."
---

# Memory/Heap Diagnostics via Roslyn MCP `diag` tool


ClrMD 3.x — programmatic memory leak hunting for any live .NET process. 22 actions, single endpoint, async snapshots.

## When to use
- User reports: "memory leak", "memory growing", "app getting slower", "dispose issue", "event handler not unsubscribed"
- Diagnosing WinForms/WPF/ASP.NET leaks (dialogs not disposed, events not unsubscribed, static caches)
- Any .NET process freeze / memory pressure investigation
- Compare heap state before/after a user action

## Do NOT use
- Non-.NET processes (native C++, Electron renderer, etc.)
- Processes you don't own (access denied)
- On the current dev VS itself (self-dump deadlock)

## Golden rule
AI never reads raw heap dumps. All work is server-side. Responses are paginated/filtered with `hints`.

## Workflow: hunt a memory leak

### 1. Find the target process
```
diag action:"find_process" processName:"Domus.Navi"
diag action:"find_process" assemblyName:"Domus.Navi.Graph"
diag action:"find_process" windowTitle:"Kalender"
```
Any combination of filters. Returns candidates with `Confidence`. Single match → use pid directly. Multiple → narrow.

### 2. Take the BEFORE snapshot
```
diag action:"snapshot_create" pid:12345 label:"before"
```
Returns `{id:"snap_001", status:"writing"}` immediately. Target process freezes during MiniDump (1-30s depending on heap size).

### 3. Poll until ready
```
diag action:"snapshot_info" snap:"snap_001"
```
Loops with `status:"writing"` showing `currentDmpSizeMb`. Eventually `status:"ready"`.

### 4. Reproduce the leak
Tell user: "Please do the suspicious action now (e.g. open and close the dialog 10 times)."

### 5. Take the AFTER snapshot
```
diag action:"snapshot_create" pid:12345 label:"after"
```

### 6. Compare — the money shot
```
diag action:"compare_snapshots" before:"snap_001" after:"snap_002" minDelta:5
```
Returns `added` / `removed` / `grew` / `shrunk` arrays. Top grower is the leaking type.

### 7. Find the retention path
```
diag action:"find_roots" snap:"snap_002" target:"GraphTerminDialogForm" maxPaths:3
```
BFS through the reverse reference graph. Returns full chains from leaking object → ... → GC root (StrongHandle / StaticVar / Thread / FinalizerQueue).

### 8. Identify the fix
Look at the `root` kind at the end of the chain:
- **StaticVar** / **Static*** — held by static field. Fix: clear the static, or use weak reference.
- **FinalizerQueue** — stuck finalizer. Fix: investigate Dispose pattern.
- **Thread** / **Stack** — held by thread stack. Fix: check async state machines, captured locals.
- **StrongHandle** — pinned (GCHandle, Handle-registered). Fix: unregister.

For WinForms-specific — almost always an **unsubscribed event**:
```
diag action:"event_subscriptions" snap:"snap_002" address:"0x1234ABCD"
```
Shows all delegate fields on the object and who's subscribed.

## Action catalogue

### Snapshot lifecycle
| Action | Required | Optional | Returns |
|---|---|---|---|
| `snapshot_create` | pid (or processName/assemblyName/typeName/windowTitle) | label | `{id, status:"writing"}` — poll `snapshot_info` |
| `snapshot_info` | snap / snapshotId / id / label | latest:true | status, size, heap, objectCount, hints |
| `snapshot_list` | — | — | all snapshots with age/size |
| `snapshot_rename` | id, newLabel | — | renamed |
| `snapshot_delete` | id | — | `{deleted:true}` |
| `snapshot_cleanup` | — | dryRun | deletes snapshots older than 24h |

### Heap analysis
| Action | Required | Optional | Purpose |
|---|---|---|---|
| `type_stats` | snap | top, page, pageSize, sortBy (size/count/name), filter | top types by size/count |
| `instance_count` | snap, typeName | — | live count + total size of a type |
| `compare_snapshots` | before, after | minDelta, filter | diff by type (added/removed/grew/shrunk) |
| `find_roots` | snap, target (typeName or 0xADDR) | maxPaths, maxDepth | BFS retention paths to GC root |

### Object inspection
| Action | Required | Optional | Purpose |
|---|---|---|---|
| `object_info` | snap, address | pageSize (max fields) | type, size, fields with values |
| `string_value` | snap, address | pageSize (max len) | extract string value |
| `array_preview` | snap, address | pageSize (limit) | array element preview |
| `event_subscriptions` | snap, address | — | delegate/EventHandler fields + their invocation lists |

### GC + WinForms
| Action | Required | Optional | Purpose |
|---|---|---|---|
| `gc_stats` | snap | — | Gen0/1/2/LOH/POH/Frozen + finalizer queue count |
| `finalizer_queue` | snap | top | top types waiting for finalization |
| `live_forms` | snap | page, pageSize | all live Form instances |
| `live_controls` | snap | page, pageSize | all live Control instances |
| `static_holders` | snap, typeName | — | static roots holding this type |

### Advanced
| Action | Required | Optional | Purpose |
|---|---|---|---|
| `trace_object` | before, after, address | — | is this object still alive in the next snapshot? |
| `find_process` | any filter | — | enumerate .NET processes |
| `leak_hunt` | — | — | returns the guided workflow |
| `cef_state` | snap | — | CefSharp-specific — live ChromiumWebBrowser instances with IsBrowserInitialized/IsDisposed/parent/handle, pinned StrongHandles to CefSharp types, `Cef.IsInitialized` static state |
| `dispose_orphans` | snap | top (maxResults) | Finds objects whose type implements IDisposable, `disposed=false`, but their parent is already `disposed=true`. Catch-all for dispose-chain bugs. |
| `force_gc` | pid (or processName/etc) | — | Composite: attach → break → Immediate `GC.Collect(2,Forced)+WaitForPendingFinalizers+GC.Collect` → continue → detach. Then call `snapshot_create` for post-GC heap. Requires VS DTE. |

## Gotchas
- **Self-dump deadlock**: do not snapshot the RoslynMcp-hosting VS itself. Snapshot external processes only.
- **Target freeze**: full memory dump pauses the target for seconds to minutes. Warn the user before snapshotting production-like apps.
- **Disk usage**: `.dmp` files are large (hundreds of MB). Run `snapshot_cleanup` regularly.
- **Labels scoped per-snapshot**: `Form#a3f1` in snap_001 is NOT the same address-wise in snap_002 (GC compacting). Use `trace_object` for cross-snapshot identity.
- **Partial info at snapshot_create return**: `heapSizeBytes` and `objectCount` are filled only after dump completes. Poll `snapshot_info` for final stats.
- **Permissions**: only processes owned by current user are attachable (`PROCESS_VM_READ | PROCESS_QUERY_INFORMATION` required).

## ⚠️ Cross-process snapshot safety

**`compare_snapshots` and `trace_object` refuse to run if `before.Pid != after.Pid`.** Snapshots from different processes (even if the same app was relaunched) cannot be compared meaningfully — heaps are different memory spaces, addresses don't carry over, type distributions differ by process state.

**Error looks like:**
```json
{
  "error": "Snapshots are from DIFFERENT processes (before pid=20424 process='Domus.Navi', after pid=19952 process='Domus.Navi'). Comparing heaps across processes is meaningless.",
  "beforePid": 20424, "afterPid": 19952,
  "hints": ["Take both snapshots of the same Domus.Navi (or whatever) process BEFORE you restart it."]
}
```

**`snapshot_list` also surfaces this**: returns `distinctPids` count + hint `⚠ Snapshots span N different process(es): pid X (count), pid Y (count). compare_snapshots / trace_object work ONLY within the same pid.`

**How to avoid:**
- Take `before` snapshot **immediately after launching the target** — don't restart the app between before/after.
- If you restarted, delete old snapshots: `diag snapshot_cleanup` or `diag snapshot_delete id:<stale>`.
- Check `diag snapshot_list` before comparing — if `distinctPids > 1`, you have mixed-process snapshots.


## ⚠️ VS debugger conflict — detach before snapshot_create

**MiniDumpWriteDump fails with `Win32 error 1 (ERROR_INVALID_FUNCTION)` if VS debugger is currently attached to the target process.**

Why: DbgHelp (used internally by MiniDumpWriteDump) interacts with the process as if it were a debugger. Windows refuses to let a second debugger operate while another one holds the debug port. If the process is also in **Break mode** (paused by VS), the snapshot cannot capture a consistent state.

**Symptoms:**
- `snapshot_create` returns `{id, status:"writing"}`
- `snapshot_info` shortly after shows `status:"failed", errorMessage:"MiniDumpWriteDump failed, Win32 error 1"`
- Happens on **all** attached processes, not just one

**Fix:** always `vs detach` **before** `diag snapshot_create`.

**Correct Force-GC + snapshot order:**
```
1. vs attach_to_process target:"<pid>" options:{break:true}
2. vs execute_immediate target:"System.GC.Collect(2, System.GCCollectionMode.Forced); System.GC.WaitForPendingFinalizers(); System.GC.Collect();"
3. vs continue                          # return target to Run mode
4. vs detach                            # RELEASE debugger port
5. diag snapshot_create pid:<pid>       # now MiniDumpWriteDump succeeds
```

If you skip step 4, snapshot will fail silently with Win32 error 1.


## Auto-cleanup on VS shutdown
When Visual Studio closes, all `.dmp` files in `.roslyn-mcp/snapshots/` are deleted automatically (`.meta.json` kept for history). No manual cleanup needed between sessions. This avoids multi-GB accumulation.

## Force GC before snapshot (via `vs` tool)
Sometimes `compare_snapshots` shows inflated deltas because Gen2 hasn't compacted yet. To force full collection in the target process before the "after" snapshot:

```
vs attach_to_process target:"<pid>" options:{break:true}
vs execute_immediate target:"System.GC.Collect(2, System.GCCollectionMode.Forced); System.GC.WaitForPendingFinalizers(); System.GC.Collect();"
vs continue
vs detach
diag snapshot_create pid:<pid> label:"after-gc"
```

This eliminates "ghost" growers that would disappear at the next natural GC. Only anything that survives forced GC is a real leak.


## Example output — compare_snapshots (leak detected)

```json
{
  "before": "snap_001",
  "after": "snap_002",
  "grew": [
    { "t": "GraphTerminDialogForm", "before": 0, "after": 10, "delta": 10, "deltaSize": 1240000 },
    { "t": "RichEditDocumentServer", "before": 2, "after": 12, "delta": 10, "deltaSize": 890000 }
  ],
  "hints": [
    "Top grower: GraphTerminDialogForm +10. Use diag find_roots snap:snap_002 target:\"GraphTerminDialogForm\" to locate.",
    "Heap delta: 14 MB, objects +2347"
  ]
}
```

## Integration with other skills
- Combine with `/roslyn-vs` for debug-time snapshot: set breakpoint → hit it → diag snapshot_create → continue
- Combine with `/roslyn-ui-test` to automate reproduction of leak scenario (click button 10x)
- After finding the leak, use `/roslyn-cs` to write the fix (Dispose override, `-=` event unsubscribe)
- Use `/roslyn-code` `find_references` to see all places a leaking type is used

## File locations
- Snapshots: `<solutionDir>/.roslyn-mcp/snapshots/snap_NNN.dmp` + `snap_NNN.meta.json`
- Plan: `DIAG-TOOL-PLAN.md` in solution root
- Architecture: server-side via ClrMD 3.1; DiagService.cs, DiagSnapshotStore.cs, McpHttpServer.Diag*.cs


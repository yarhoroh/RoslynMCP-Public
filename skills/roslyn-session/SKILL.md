---
name: roslyn-session
description: "Roslyn MCP session and memory management — start, record, restore, end, maintenance, change graph.\nTRIGGER when: session management, memory maintenance, change graph tracking, consolidate/audit/rebalance.\nDO NOT TRIGGER when: just recalling memory or simple remember/forget."
---

# Roslyn MCP: Sessions & Memory

> **Language rule:** Write all memory entries in English. Respond to the user in whatever language they used.

## Session lifecycle

```
start → remember/learn during work → end (with summary)
```

### Start

```json
memory_start_session { "goal": "<sessionGoal>" }
// → returns sessionId — keep it for the rest of the session
```

### End

```json
memory_end_session {
  "sessionId": "<sessionId>",
  "summary": "<whatWasDone>"
}
```

### List / Restore / Forget

```json
// List sessions
memory_list_sessions { "limit": 10, "status": "active" }
// status: "active" | "completed" | "abandoned"

// Restore context from past session
memory_restore { "sessionId": "<sessionId>", "includeGraph": true }

// Forget session memories (keep global by default)
memory_forget_session {
  "sessionId": "<sessionId>",
  "keepGlobal": true,
  "deleteSession": false
}
```

---

## Memory CRUD

### Save

```json
// Save a decision, change, error, fix
memory_remember {
  "type": "Decision",
  "key": "<key>",
  "content": "<content>",
  "scope": "session",
  "importance": 5,
  "tags": ["<tag1>", "<tag2>"],
  "files": ["<filePath>"],
  "parentId": "<parentMemoryId>",
  "relatedTo": ["<relatedMemoryId>"]
}
// type: Decision | Change | Error | Fix | Pattern | Lesson
// scope: "session" (default) | "global" (only for reusable cross-session facts)

// Save a lesson/pattern
memory_learn {
  "key": "<key>",
  "content": "<content>",
  "scope": "session",
  "importance": 5,
  "tags": ["<tag1>"],
  "files": ["<filePath>"]
}
```

### Read

```json
// Recall with filters
memory_recall {
  "key": "<key>",
  "types": ["Decision", "Lesson"],
  "tags": ["<tag>"],
  "files": ["<filePath>"],
  "scope": "global",
  "sessionId": "<sessionId>",
  "limit": 10
}

// Full-text search
memory_search { "query": "<searchText>", "types": ["Decision"], "limit": 20 }

// Get context for a task/file (combines global + session memories)
memory_context {
  "forTask": "<taskDescription>",
  "forFile": "<filePath>",
  "forKey": "<fuzzyKey>",
  "sessionId": "<sessionId>",
  "types": ["Decision", "Pattern"],
  "limit": 10
}
```

### Update

```json
memory_update {
  "memoryId": "<memoryId>",
  "content": "<newContent>",
  "importance": 8,
  "addTags": ["<newTag>"],
  "removeTags": ["<oldTag>"]
}
```

### Delete

```json
memory_forget { "memoryId": "<memoryId>" }
```

### Promote (session → global)

```json
memory_promote {
  "memoryId": "<memoryId>",
  "targetScope": "global",
  "reason": "<whyPromoting>",
  "dryRun": false
}
```

---

## Memory types reference

| Type | When to use |
|------|-------------|
| `Decision` | "Decided to use X instead of Y" |
| `Change` | "Added method Z", "Removed class W" |
| `Error` | "Build error in file X" |
| `Fix` | "Fixed by changing Y" (link via `parentId` to Error) |
| `Pattern` | "Always check null before calling X" |
| `Lesson` | "Learned that X works better than Y" |

---

## Scope rules

| Scope | Visibility | When to use |
|-------|-----------|-------------|
| `session` | Current session only | Temporary context, current task decisions |
| `global` | Always visible | Reusable rules, patterns, cross-session facts |

**Default scope is `session`**. Use `global` only when user explicitly says "always", "remember forever", "global rule".

---

## Memory maintenance

```json
// Health statistics
memory_analyze {}

// Find duplicates (dry run first!)
memory_consolidate { "threshold": 0.8, "dryRun": true }

// Audit scope quality — suggest corrections
memory_audit_scopes { "limit": 200, "onlyGlobal": true, "confidenceThreshold": 0.65 }

// Apply audit suggestions
memory_rebalance_scopes { "limit": 200, "dryRun": true }
```

---

## Change graph

Track file changes, dependencies, and cause-effect relationships during development.

```json
// Record a file change
graph_track_change {
  "filePath": "<filePath>",
  "description": "<whatChanged>",
  "changeType": "modify"
}
// changeType: "add" | "modify" | "delete" | "refactor"

// Record a dependency between symbols
graph_track_dependency {
  "fromSymbol": "<fromSymbol>",
  "toSymbol": "<toSymbol>",
  "dependencyType": "uses"
}
// dependencyType: "uses" | "inherits" | "implements" | "calls"

// Record cause-effect relationship
graph_track_cause {
  "causeNodeId": "<causeNodeId>",
  "effectDescription": "<whatHappened>",
  "effectType": "Error"
}
// effectType: "Error" | "Fix" | "Change" | "Decision"

// Impact analysis from graph
graph_get_impact { "nodeId": "<nodeId>", "depth": 2 }

// Change history for file or symbol
graph_get_history { "targetName": "<name>", "targetType": "File", "limit": 20 }
// targetType: "File" | "Symbol"

// Delete a graph node
graph_delete_node { "nodeId": "<nodeId>", "deleteOrphans": false }

// Visualize the graph
graph_visualize { "format": "mermaid", "rootNodeId": "<startNode>", "depth": 2 }
// format: "mermaid" | "dot"
```

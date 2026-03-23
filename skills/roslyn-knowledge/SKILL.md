---
name: roslyn-knowledge
description: "Roslyn MCP knowledge base — KB vs Memory, CRUD, search, hierarchy, categories.\nTRIGGER when: ANY knowledge base operation — kb_add, kb_search, kb_list, kb_tree, kb_update, kb_delete.\nDO NOT TRIGGER when: working with session memory (memory_*) or code analysis."
---

# Roslyn MCP: Knowledge Base (KB)

> **Language rule:** Write all KB entries in English. Respond to the user in whatever language they used.

## KB vs Memory

| Criterion | Memory | KB |
|---|---|---|
| Purpose | Working context, decisions, lessons | Permanent reference documentation |
| Lifetime | Session / Global (can be purged) | Forever |
| Written by | AI during work | Intentionally, like a document |
| Structure | key/value + tags | Tree with parentId |

**Rule:** needed once → Memory. Needed again in a month → KB.

---

## CRUD

### Add (all 4 params required)

```json
kb_add {
  "type": "<type>",
  "category": "<category>",
  "title": "<title>",
  "content": "<markdownContent>",
  "parentId": "<parentId>",
  "tags": ["<tag1>", "<tag2>"],
  "importance": 5,
  "status": "active",
  "url": "<url>",
  "reviewAt": "2025-06-01",
  "relatedTo": ["<relatedEntryId>"]
}
```

| Required param | Values |
|---|---|
| `type` | Note, Idea, Doc, Snippet, Link, History, Todo, Reference |
| `category` | project, tech, process, resource |
| `title` | short descriptive title |
| `content` | markdown content |

| Optional param | Description |
|---|---|
| `parentId` | parent entry ID for hierarchy |
| `tags` | array of searchable tags |
| `importance` | 1-10 (default 5) |
| `status` | active, someday, archived, pinned |
| `url` | URL (for Link type) |
| `reviewAt` | ISO date to resurface entry |
| `relatedTo` | array of related entry IDs |

### Get

```json
kb_get { "id": "<id>", "includeRelated": true }
```

### Update (only passed fields are changed)

```json
kb_update {
  "id": "<id>",
  "title": "<newTitle>",
  "content": "<newContent>",
  "importance": 8,
  "status": "pinned",
  "category": "tech",
  "type": "Doc",
  "parentId": "<newParentId>",
  "url": "<newUrl>",
  "reviewAt": "2025-07-01",
  "addTags": ["<newTag>"],
  "removeTags": ["<oldTag>"],
  "addRelated": ["<relatedId>"],
  "removeRelated": ["<oldRelatedId>"]
}
```

### Delete

```json
kb_delete { "id": "<id>", "cascade": false }
// cascade: true = also delete children
```

---

## Search & List

```json
// Full-text + vector search (returns summaries by default)
kb_search {
  "query": "<searchQuery>",
  "type": "Doc",
  "category": "tech",
  "tags": ["<tag>"],
  "status": "active",
  "createdAfter": "2025-01-01",
  "createdBefore": "2025-12-31",
  "includeContent": false,
  "limit": 10
}

// List with filters (returns summaries by default)
kb_list {
  "type": "Doc",
  "category": "tech",
  "tags": ["<tag>"],
  "status": "active",
  "parentId": "<parentId>",
  "createdAfter": "2025-01-01",
  "createdBefore": "2025-12-31",
  "includeContent": false,
  "limit": 20,
  "orderBy": "importance"
}
// orderBy: "importance" (default) | "created" | "updated"

// Related entries
kb_related {
  "id": "<id>",
  "relation": "relates_to",
  "includeVector": false,
  "limit": 10
}
// relation: "relates_to" | "extends" | "implements" | "supersedes"
```

> **Token optimization:** `includeContent: false` (default) returns titles only. Use `kb_get` for full content of specific entries.

---

## Hierarchy

```json
// Get knowledge tree
kb_tree { "rootId": "<rootId>", "depth": 3 }
// rootId: null = all roots; depth: 1-10

// Create a child node
kb_add {
  "type": "Doc",
  "category": "tech",
  "title": "<childTitle>",
  "content": "<content>",
  "parentId": "<parentId>"
}
```

Think of the tree like folders: module at the top, components inside.

---

## Recommended categories

| Category | What to store |
|---|---|
| `project` | Project structure, modules, layers |
| `tech` | Interfaces, DTOs, APIs, patterns |
| `process` | Conventions, naming, coding rules |
| `resource` | External links, tools, documentation |

---

## Type reference

| Type | When to use |
|---|---|
| `Note` | General observations, notes |
| `Idea` | Future improvements, proposals |
| `Doc` | Architecture, module docs, guides |
| `Snippet` | Code examples, templates |
| `Link` | External URLs with context |
| `History` | Decision log, change history |
| `Todo` | Tasks, action items |
| `Reference` | API contracts, schemas |

---

## KB vs Memory — decision guide

```
KB → when:
  ✓ Describing the architecture of a module
  ✓ Interface contract used by multiple teams
  ✓ Convention that must always be followed
  ✓ History of an important architectural decision
  ✓ Code snippet that will be reused

Memory → when:
  ✓ "Currently working on task X"
  ✓ "Approach for current refactoring"
  ✓ "Learned that X works better than Y"
  ✓ "Bug found: null reference in method Z"
```

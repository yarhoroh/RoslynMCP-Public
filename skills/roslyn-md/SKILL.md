---
name: roslyn-md
description: "Structured .md file editing via Roslyn MCP `md` tool — toc, read, edit, insert, append, delete, search sections.\nTRIGGER when: ANY operation on ANY .md file — read, edit, search, create, delete sections, view structure.\nDO NOT TRIGGER when: working with non-.md files."
---

# Markdown File Editing via Roslyn MCP

## Golden Rule: NEVER use Read/Edit/Write for .md files. Always use `md` tool.

## Single-File Workflow

### 1. Get structure
```
md action:"toc" filePath:"README.md"           → flat heading list with lines
md action:"tree" filePath:"README.md"           → hierarchical tree with children[] and treePath
```

### 2. Search by keyword
```
md action:"search" filePath:"README.md" query:"17.4"
```
Fuzzy matching, BM25-like scoring. Searches headings (3x weight) and body.

### 3. Read specific section
```
md action:"read_section" filePath:"README.md" section:"Installation"
```

### 4. Read by line range
```
md action:"read" filePath:"README.md" line:50 count:20
```

### 5. Edit section (replace body, keep heading)
```
md action:"edit_section" filePath:"README.md" section:"Installation" content:"new body"
```

### 6. Insert new section
```
md action:"insert_section" filePath:"README.md" section:"### New Section" after:"Features" content:"body"
```

### 7. Append to section
```
md action:"append" filePath:"README.md" section:"Installation" content:"- new item"
```

### 8. Delete section
```
md action:"delete_section" filePath:"README.md" section:"Old Version"
```

### 9. Simple text replace
```
md action:"replace" filePath:"README.md" old:"120+ tools" content:"130+ tools"
```

## Tree-First Document Creation

Create documents as a tree — no need to think about `#` vs `##` vs `###`.

### Create new document
```
md action:"create_doc" filePath:"docs/guide.md" title:"User Guide" content:"Introduction text."
```

### Add section under parent (auto heading level)
```
md action:"add_node" filePath:"docs/guide.md" section:"Authentication" parent:"User Guide" content:"OAuth2 flow..."
md action:"add_node" filePath:"docs/guide.md" section:"Token Refresh" parent:"Authentication" content:"..."
```
Result: `# User Guide` → `## Authentication` → `### Token Refresh` — levels automatic.

### Bulk create entire document in one call
```
md action:"bulk_create" filePath:"docs/api.md" content:"API Reference\n  Auth\n    OAuth: OAuth2 flow\n    Keys: API key management\n  Endpoints\n    Users: CRUD operations\n    Orders: Order lifecycle"
```
2 spaces per indent level. `Heading: body` format. Creates full hierarchy.

### Rename, move sections
```
md action:"rename_node" filePath:"doc.md" section:"Old Name" content:"New Name"
md action:"move_node" filePath:"doc.md" section:"Auth" parent:"Security"    → moves + adjusts heading levels
```

## Multi-File Indexing & Search (FTS5)

Index all .md files in a directory. SQLite FTS5 full-text search with BM25 ranking.

### Index a directory
```
md action:"index" path:"C:\docs"       → indexes all .md recursively
md action:"index" path:"C:\docs" force:true   → force re-index all
```
On-demand: only re-indexes changed files (lastModified check). Deletes removed files from index.

### Search across all files
```
md action:"search_all" query:"OAuth authentication" path:"C:\docs"
```
Returns: file, heading, treePath, startLine, endLine, score, preview. Prefix matching (`OAuth` finds `OAuth2`). Works with any language (Russian, Ukrainian, Chinese, etc.).

### Auto-indexing
`search_all` auto-indexes the directory on first call. No need to call `index` separately.

### Statistics
```
md action:"stats" path:"C:\docs"   → files, sections, totalSize, dbPath
```

## Key Rules
- `filePath` — for single-file actions (absolute or relative to solution)
- `path` — for multi-file actions (directory)
- `section` matching: case-insensitive, substring, normalized
- `parent` in add_node/move_node: auto-determines heading level
- Section = heading + content until next heading of same/higher level
- `tree` returns hierarchical view, `toc` returns flat list
- Index stored in `.roslyn-mcp/md_index.db` (SQLite + FTS5 via winsqlite3.dll)
- NEVER use Read/Edit/Write for .md files — always `md` tool via Roslyn


### Auto-Reindex on Edit

All edit operations (edit_section, add_node, append, replace, etc.) automatically re-index the modified file. Index is always up-to-date after edits through md tool.

## When to use
- Any .md file operation: read, edit, create, delete, search
- Multi-file documentation search across projects
- Structured document creation (tree-first approach)
- Updating changelogs, READMEs, skill files
- Searching large documentation sets


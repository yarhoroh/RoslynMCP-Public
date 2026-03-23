---
name: roslyn-md
description: "Structured .md file editing via Roslyn MCP `md` tool — toc, read, edit, insert, append, delete, search sections.\nTRIGGER when: ANY operation on ANY .md file — read, edit, search, create, delete sections, view structure.\nDO NOT TRIGGER when: working with non-.md files."
---

# Markdown File Editing via Roslyn MCP

## Golden Rule: NEVER read full .md file. Always use `md` tool.

## Workflow

### 1. Get structure (TOC)
```
md action:"toc" filePath:"README.md"
```
Returns: all headings with line numbers and nesting levels. This is your "tree".

### 2. Search by keyword (find sections by content)
```
md action:"search" filePath:"README.md" query:"17.4"
```
Returns: matching sections ranked by relevance (BM25-like scoring). Searches BOTH headings and body text.
- Fuzzy matching: "17.4" finds "Some Section — Title"
- Returns: heading, startLine, endLine, score, snippet
- Use `maxResults:5` to limit results

### 3. Read specific section
```
md action:"read_section" filePath:"README.md" section:"Installation"
```
Returns: only that section's content with line numbers. Section match is case-insensitive substring with normalized matching.
- "17.4" matches "Some Section — Full Title Here"
- Strips markdown formatting for comparison

### 4. Read by line range (for raw line access)
```
md action:"read" filePath:"README.md" line:50 count:20
```

### 5. Edit existing section (replace body, keep heading)
```
md action:"edit_section" filePath:"README.md" section:"Installation" content:"new body text here"
```
**Note:** For long content, use Claude's Edit tool directly (JSON has size limits).

### 6. Insert new section
```
md action:"insert_section" filePath:"README.md" section:"### New Section Title" after:"Features" content:"- item 1\n- item 2"
```
`section` = the new heading line. `after` = insert after this existing heading.

### 7. Append to section (add to end without replacing)
```
md action:"append" filePath:"README.md" section:"Installation" content:"- **New:** another feature"
```

### 8. Delete section
```
md action:"delete_section" filePath:"README.md" section:"Old Version"
```

### 9. Simple text replace
```
md action:"replace" filePath:"README.md" old:"120+ tools" content:"130+ tools"
```

## Key Rules
- `filePath` can be relative (to solution dir) or absolute
- `section` matching: case-insensitive, substring, normalized — "17.4" matches "Some Section — Title"
- `search` searches both headings AND body content, returns ranked results
- Section = heading + all content until next heading of same/higher level
- `edit_section` keeps the heading line, replaces everything below it
- `insert_section` needs `section` (the new heading WITH ##) and `after` (existing heading to insert after)
- Always `toc` first to orient, then `search` or targeted reads/edits
- For long content edits, prefer Claude's Edit tool over `edit_section` (avoids JSON size limits)
- NEVER use Read/Edit for .md files, always use md tool via Roslyn

## When to use

- Updating README with new version/features
- Editing CLAUDE.md instructions
- Editing skill files (SKILL.md) via md tool
- Working with any documentation file
- Updating changelogs
- Reading large .md files without loading full context
- Searching for specific content across large documents


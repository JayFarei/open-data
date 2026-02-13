# Persistence Specification: [Project Name]

> Version: 0.1.0
> Date: [YYYY-MM-DD]
> Author: [Name]
> Status: Draft | Review | Approved

## Overview

[One paragraph describing the system, its purpose, and why an open data
persistence layer is the right choice. State the primary design constraint
(e.g., "data must be fully readable without the application").]

---

## 1. Entity Model

### 1.1 Entity Types

| Type | File Pattern | Volume | Creation | Lifecycle |
|------|-------------|--------|----------|-----------|
| [e.g., Bookmark] | `feed/YYYY-MM-DD_slug.md` | ~1000/year | Automated | Append-only |
| [e.g., Project] | `projects/slug.md` | ~50 | Manual | Edited frequently |
| [e.g., Person] | `people/name.md` | ~200 | Manual | Rarely edited |

### 1.2 Entity Schemas

#### [Entity Type 1]

```yaml
---
# Mandatory
type: [type-name]
title: [string]
created: [datetime]

# Recommended
tags: [list]
status: [enum: draft | active | archived]
source: [text]

# Optional
[domain-specific fields]
---

[Body content description — what goes in the Markdown body vs. frontmatter]
```

#### [Entity Type 2]

[Repeat for each entity type]

### 1.3 Property Type Registry

```json
{
  "types": {
    "type": "text",
    "title": "text",
    "created": "datetime",
    "tags": "tags",
    "status": "text",
    "source": "text",
    "aliases": "aliases"
  }
}
```

Location: `.config/types.json`

---

## 2. Directory Structure

```
[vault-name]/
├── .config/                    # Application configuration
│   ├── app.json                # Core settings
│   ├── types.json              # Property type registry
│   └── plugins/                # Plugin/extension settings
├── .index/                     # Derived data (regenerable, gitignored)
│   └── [index files]
├── [collection-1]/             # Entity collection
│   └── [files]
├── [collection-2]/
│   └── [files]
├── _templates/                 # File templates for each entity type
│   └── [template files]
├── _schemas/                   # Schema definitions (optional)
│   └── [schema files]
└── .gitignore
```

### 2.1 Naming Conventions

| Context | Convention | Example |
|---------|-----------|---------|
| Entity files | `[slug].md` or `[date]_[slug].md` | `project-alpha.md` |
| Slugs | lowercase, hyphens, no spaces | `my-project-name` |
| Temporal files | ISO 8601, hyphens for time | `2025-01-15T10-30-00` |
| Collections | plural, lowercase, hyphens | `daily-notes/` |
| Config | dotfile directory | `.config/` |
| Index | dotfile directory | `.index/` |
| Templates | underscore prefix | `_templates/` |

### 2.2 File System Rules

- Maximum path depth: [N] levels from vault root
- Maximum file name length: [N] characters (recommend ≤100)
- Character restrictions: [list forbidden characters beyond OS limits]
- Case sensitivity: [case-insensitive matching for links, case-preserving for display]

---

## 3. Relationship Model

### 3.1 Link Syntax

| Syntax | Meaning | Example |
|--------|---------|---------|
| `[[target]]` | Basic internal link | `[[project-alpha]]` |
| `[[target\|display]]` | Link with display text | `[[project-alpha\|Alpha]]` |
| `[[target#Heading]]` | Link to heading | `[[project-alpha#Timeline]]` |
| `[[target#^block-id]]` | Link to block | `[[project-alpha#^budget]]` |
| `![[target]]` | Embed/transclusion | `![[project-alpha]]` |

### 3.2 Typed Relationships

| Property | Cardinality | Meaning |
|----------|------------|---------|
| [e.g., `depends-on`] | Many | [This entity depends on the targets] |
| [e.g., `owned-by`] | One | [The person responsible] |
| [e.g., `related`] | Many | [General association] |

### 3.3 Link Resolution

**Algorithm:** [Shortest path / Relative / Absolute]

**Resolution rules:**
1. Strip `.md` extension from link target if present
2. Search vault for files matching the target (case-insensitive)
3. If exactly one match: resolve
4. If multiple matches: require path disambiguation
5. If no match: mark as unresolved link

**Alias resolution:** [Yes/No] — If yes, also match against `aliases:` values

**Automatic link updates on rename:** [Yes/No/Configurable]

---

## 4. Tag System

### 4.1 Syntax

- Inline: `#tag-name` or `#parent/child`
- Frontmatter: `tags: [tag-name, parent/child]`
- Nesting depth: [Maximum levels, e.g., 3]

### 4.2 Tag Naming Rules

- Allowed characters: [letters, numbers, hyphens, underscores, forward slashes]
- Must contain at least one non-numeric character
- Case-insensitive for search, case-preserving for display

### 4.3 Reserved Tag Prefixes

| Prefix | Meaning |
|--------|---------|
| [e.g., `status/`] | [Lifecycle state: status/active, status/archived] |
| [e.g., `type/`] | [Entity classification] |
| [e.g., `source/`] | [Origin system: source/twitter, source/github] |

---

## 5. Index Specification

### 5.1 Index Storage

- Format: [JSON / SQLite / In-memory]
- Location: `.index/`
- Rebuild trigger: [On startup / On file change / Manual command]

### 5.2 Indexed Fields

| Field | Source | Purpose |
|-------|--------|---------|
| File path | File system | Unique identifier |
| Frontmatter | YAML parsing | Structured queries |
| Internal links | Body parsing | Graph construction |
| Tags | Frontmatter + inline | Categorization queries |
| Headings | Body parsing | Section navigation |
| Block IDs | Body parsing | Block reference resolution |
| File timestamps | File system | Temporal queries |

### 5.3 Derived Structures

**Resolved link map:**
```json
{ "source-path": { "target-path": count } }
```

**Backlink map:** Computed as inverse of resolved link map.

**Tag index:**
```json
{ "tag-name": ["file-path-1", "file-path-2"] }
```

---

## 6. Non-Markdown Content

### 6.1 Supported File Types

| Category | Extensions | Storage location |
|----------|-----------|-----------------|
| Images | `.png`, `.jpg`, `.svg`, `.gif`, `.webp` | `[location]` |
| Documents | `.pdf` | `[location]` |
| Data | `.csv`, `.json` | `[location]` |

### 6.2 Asset Metadata Strategy

[Describe: centralized / co-located / sidecar — and the metadata schema for assets]

### 6.3 Embed Syntax

```
![[image.png]]              Image display
![[image.png|300]]          Image with width
![[document.pdf]]           PDF viewer
![[document.pdf#page=3]]    PDF at page
```

---

## 7. Configuration Separation

### 7.1 Config Directory Structure

```
.config/
├── app.json          # [Describe contents]
├── types.json        # Property type registry
├── plugins/          # Per-plugin settings
│   └── [plugin-id]/
│       └── data.json
└── [other config files]
```

### 7.2 What Is NOT Config

These belong in user content, not `.config/`:
- Templates (they are user-authored content)
- Schema definitions (they describe user data)
- Index/derived data (belongs in `.index/`)

---

## 8. Version Control Integration

### 8.1 .gitignore

```
# Derived data (always regenerable)
.index/

# Ephemeral application state
.config/workspace.json

# OS artifacts
.DS_Store
Thumbs.db
*.tmp
```

### 8.2 Tracked Files

- All `.md` files (user content)
- All assets (images, PDFs, etc.)
- `.config/` (minus ephemeral state)
- `_templates/`, `_schemas/`

---

## 9. Portability Assessment

### 9.1 Standard Features Used

[List Markdown features used that are CommonMark/GFM standard]

### 9.2 Non-Standard Extensions Required

| Extension | Degradation behavior in unsupported tools |
|-----------|------------------------------------------|
| Wikilinks `[[]]` | Displayed as literal text `[[target]]` |
| Embeds `![[]]` | Displayed as literal text |
| Block refs `#^id` | Displayed as literal text |
| Callouts `> [!type]` | Displayed as plain blockquote |
| Highlights `==text==` | Displayed as literal `==text==` |
| Comments `%% %%` | Displayed as literal text |

### 9.3 Migration Escape Hatch

To migrate away from this system:
1. Copy the vault directory
2. [Any conversion steps needed for non-standard syntax]
3. Frontmatter remains valid YAML in any tool
4. Content remains valid Markdown (possibly with unresolved syntax)

Estimated migration effort: [Hours / complexity]

---

## 10. Appendices

### A. Example Files

[Include 2-3 complete example files showing the schema in practice]

### B. Validation Rules

[If using automated validation, list the rules and how to run them]

### C. Changelog

| Version | Date | Changes |
|---------|------|---------|
| 0.1.0 | [date] | Initial draft |

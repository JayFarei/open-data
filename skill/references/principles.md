# Futureproof Persistence Principles

These are non-negotiable. Every design decision must be evaluated against them.

## Principle 1: Files Over Databases

The canonical data store is the file system. If the app disappears tomorrow, the
user's data must remain fully readable and useful with nothing more than a text
editor and a file browser.

**Test:** Can you `ls` and `cat` your way to understanding the data?

## Principle 2: Convention Over Format

Structure emerges from naming conventions, directory hierarchies, YAML frontmatter,
and inline syntax — not from proprietary binary formats, custom databases, or
app-specific encodings.

**Test:** Does a new developer understand the schema by looking at the folder structure?

## Principle 3: Indexing Over Storage

Derived data (graphs, search indices, caches, computed views) must always be
regenerable from the source files. The index accelerates; it never becomes the authority.

**Test:** Can you delete every derived file and rebuild without data loss?

## Principle 4: Graceful Degradation

When a feature isn't supported by a consuming tool, the data should degrade to
something readable rather than becoming opaque. A wikilink in a non-wiki editor
is still readable text. A YAML block in a dumb editor is still key-value pairs.

**Test:** Open a data file in Notepad. Is it still useful?

## Principle 5: Separation of Concerns

Application state (UI preferences, plugin configs, workspace layout) lives in a
clearly separated directory (like `.obsidian/` or `.config/`). User data never
co-mingles with app config in the same files.

**Test:** Can you `rm -rf .config/` without losing any user data?

## Principle 6: Tool-Agnostic Identity

Files are identified by their path, not by UUIDs, internal IDs, or database keys.
The file name IS the identity. Moving or renaming a file changes its identity —
and the system must handle that explicitly.

**Test:** Does any file reference an opaque ID that has no meaning outside the app?

## Principle 7: Composability

The data model must be layerable. Each layer is independently valuable:

- **Layer 0:** Files exist and are readable plain text
- **Layer 1:** YAML frontmatter adds structured metadata
- **Layer 2:** Wikilinks add relationships between files
- **Layer 3:** Indices add queryability (search, graph, backlinks)
- **Layer 4:** Plugins add domain-specific features

**Test:** Does each layer add value without requiring the layers above it?

---

## Design Vocabulary

Use these terms consistently across all outputs:

| Term | Definition |
|------|-----------|
| **Vault** | A root directory serving as the boundary for a data collection |
| **Note / Document** | A single `.md` file representing one entity or record |
| **Frontmatter** | YAML metadata block at the top of a document |
| **Property** | A key-value pair in frontmatter, analogous to a database column |
| **Wikilink** | `[[target]]` syntax creating a relationship between documents |
| **Backlink** | The inverse of a wikilink — computed, not stored |
| **Embed / Transclusion** | `![[target]]` syntax for inline content inclusion |
| **Block reference** | `#^id` syntax for linking to a specific paragraph |
| **Tag** | `#category` or `#category/sub` for hierarchical classification |
| **Index / Cache** | Derived data structure regenerable from source files |
| **Config directory** | Hidden directory for app-level settings, separate from data |
| **Schema enforcement** | Type constraints on properties, applied vault-wide |
| **Link resolution** | Algorithm for turning a link target into a file path |
| **Graceful degradation** | Property of data that remains useful in unsupported tools |

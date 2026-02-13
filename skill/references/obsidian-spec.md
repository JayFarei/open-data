# Obsidian Open Data Model — Technical Reference

This is the condensed technical reference for Obsidian's data model. It serves as
the foundational example of a futureproof persistence system. The full specification
with detailed research and sourcing was produced separately; this document extracts
the implementable details.

---

## Architecture Summary

Obsidian treats a local folder of Markdown files as a queryable knowledge database.
There is no proprietary database, no binary format, no cloud dependency. The file
system is the canonical data store. A MetadataCache (IndexedDB) accelerates queries
but is always regenerable from files.

---

## Vault Structure

A vault = a directory. The `.obsidian/` subdirectory holds app config. Everything
else is user content.

```
MyVault/
├── .obsidian/              # App config (hidden)
│   ├── app.json            # Editor settings, link behavior
│   ├── appearance.json     # Theme, fonts
│   ├── types.json          # Property type registry
│   ├── core-plugins.json   # Core plugin toggle map
│   ├── community-plugins.json  # Enabled plugin ID list
│   ├── hotkeys.json        # Custom hotkey overrides only
│   ├── workspace.json      # Ephemeral UI state (don't sync)
│   ├── graph.json          # Graph view settings
│   ├── plugins/            # Community plugin code + settings
│   ├── themes/             # Community themes
│   └── snippets/           # User CSS snippets
├── [user content folders and files]
└── [attachments]
```

**Identity:** File path relative to vault root = unique identifier. File name minus
`.md` = note title. `aliases:` frontmatter provides alternative names.

**Supported formats:** `.md` (editable notes), `.canvas` (JSON Canvas), images
(avif/bmp/gif/jpeg/jpg/png/svg/webp), audio (flac/m4a/mp3/ogg/wav/webm/3gp),
video (mkv/mov/mp4/ogv/webm), `.pdf`.

---

## Markdown Flavor

**Base:** CommonMark + GitHub Flavored Markdown (tables, strikethrough, task lists, autolinks)

**Standard features:** Headings, bold, italic, code blocks (PrismJS), blockquotes,
lists, task checkboxes, footnotes, GFM tables, LaTeX math ($/$$ via MathJax),
Mermaid diagrams in fenced code blocks.

**Non-standard default:** Single newlines create line breaks (toggleable via
Strict Line Breaks setting).

**Obsidian-specific extensions (OFM):**

| Feature | Syntax | Standard? |
|---------|--------|-----------|
| Wikilinks | `[[target]]` | ❌ OFM |
| Display text | `[[target\|display]]` | ❌ OFM |
| Heading links | `[[target#Heading]]` | ❌ OFM |
| Block references | `[[target#^block-id]]` | ❌ OFM |
| Embeds | `![[target]]` | ❌ OFM |
| Image resize | `![[img.png\|300]]` or `\|640x480` | ❌ OFM |
| Highlights | `==text==` | ❌ OFM |
| Comments | `%% hidden %%` | ❌ OFM |
| Callouts | `> [!type]` in blockquote | ❌ OFM |

**Callout types:** note, abstract, info, todo, tip, success, question, warning,
failure, danger, bug, example, quote. Foldable: `> [!type]-` (collapsed),
`> [!type]+` (expanded).

---

## YAML Frontmatter / Properties

Delimited by `---` on line 1. Standard YAML syntax. Property names are case-insensitive.

**Type system (Obsidian 1.4+):**

| UI Type | Internal type | YAML example |
|---------|--------------|-------------|
| Text | `text` | `key: "value"` |
| List | `multitext` | `key: [a, b, c]` |
| Number | `number` | `key: 42` |
| Checkbox | `checkbox` | `key: true` |
| Date | `date` | `key: 2025-01-15` |
| Date & Time | `datetime` | `key: 2025-01-15T14:30` |
| Tags | `tags` | (reserved for `tags` property) |
| Aliases | `aliases` | (reserved for `aliases` property) |

**Type enforcement:** A property name has one type vault-wide. Stored in
`.obsidian/types.json`. Unregistered properties default to `text`.

**Reserved properties:** `tags` (categorization, no `#` prefix in YAML),
`aliases` (alternative names for link resolution), `cssclasses` (CSS classes
for note styling). Singular forms deprecated since 1.4.

**Wikilinks in frontmatter:** Must be quoted for valid YAML: `"[[target]]"`.
Recognized by Obsidian's link resolution and graph.

**Search integration:** `[property:value]` syntax in search queries.

---

## Internal Linking

### Wikilink Syntax
```
[[Note Name]]                     Basic link
[[Note Name|Display Text]]        Custom display text  
[[Note Name#Heading]]             Heading link
[[#Heading]]                      Same-note heading link
[[Note Name#^block-id]]           Block reference
![[Note Name]]                    Embed (transclusion)
```

Non-Markdown files require extension: `[[image.png]]`, `[[report.pdf]]`.

### Block IDs
Appended to paragraphs: `This paragraph. ^my-block-id`
Characters: Latin letters, numbers, dashes. Auto-generated: 6-char alphanumeric.

### Link Resolution Modes
1. **Shortest path** (default): Name only if unique, full path if ambiguous
2. **Relative path**: `../` notation from current file
3. **Absolute path**: Full vault-relative path, no leading slash

**Case-insensitive.** `.md` extension optional. Automatic link update on file rename.
Heading renames do NOT update links (known limitation).

### Standard Markdown Links
Also supported: `[display](path/to/note.md)`. Requires `.md` extension and
`%20` for spaces. Default format when "Use [[Wikilinks]]" is off.

---

## Backlinks and Graph

### Data Structure
```typescript
resolvedLinks: Record<string, Record<string, number>>
// { "notes/A.md": { "notes/B.md": 2 } }

unresolvedLinks: Record<string, Record<string, number>>
// { "notes/A.md": { "NonExistent": 1 } }
```

**Backlinks** = inverse lookup of resolvedLinks. **Unlinked Mentions** = case-insensitive
text matches of filename/aliases not wrapped in `[[]]`.

**Graph View:** Force-directed WebGL layout. Node size ∝ incoming links. Filterable
by path, tag, and search query. Tags optionally rendered as nodes.

---

## Tags

**Syntax:** `#tag` inline, `#parent/child` nested, `tags: [tag]` in frontmatter.
**Rules:** Letters, numbers, underscores, hyphens, forward slashes. At least one
non-numeric character. Case-insensitive for search.
**Hierarchy:** Searching parent matches all descendants.
**API:** `getAllTags(cache)` merges inline + frontmatter tags.

---

## Embeds / Transclusion

```
![[note]]                    Full note
![[note#Heading]]            Heading section (to next equal/higher heading)
![[note#^block-id]]          Single block
![[image.png]]               Image
![[image.png|300]]           Image with width
![[image.png|640x480]]       Image with dimensions
![[doc.pdf]]                 PDF viewer
![[doc.pdf#page=3]]          PDF at page
![[audio.mp3]]               Audio player
![[video.mp4]]               Video player
```

Standard Markdown images also supported with resize: `![alt|300](url)`.

---

## Obsidian URI Scheme

**Format:** `obsidian://action?param=value`

**Actions:**
- `open` — `vault`, `file`, `path` params
- `search` — `query` param
- `new` — `name`, `content`, `append`, `overwrite` params
- `hook-get-address` — returns Markdown link to current note

**Shorthands:** `obsidian://vault/name/note`, `obsidian:///absolute/path`

---

## Plugin Data Strategies

Plugins store data using four approaches, all preserving the Markdown foundation:
1. **YAML frontmatter** — Page-level metadata
2. **Inline Markdown** — Emoji syntax (Tasks), `Key:: Value` (Dataview)
3. **Fenced code blocks** — Query languages (` ```dataview `, ` ```tasks `)
4. **Comment blocks** — Hidden config (`%% plugin:settings {...} %%`)

Plugin settings persist in `.obsidian/plugins/<id>/data.json`.

---

## MetadataCache API

```typescript
interface CachedMetadata {
  links?: LinkCache[];
  embeds?: EmbedCache[];
  tags?: TagCache[];
  headings?: HeadingCache[];
  sections?: SectionCache[];
  listItems?: ListItemCache[];
  frontmatter?: FrontMatterCache;
  frontmatterLinks?: FrontmatterLinkCache[];
  blocks?: Record<string, BlockCache>;
}
```

All items include source positions (line, col, offset). Key methods:
- `getFirstLinkpathDest(linkpath, sourcePath)` — resolve link to file
- `getFileCache(file)` — get CachedMetadata for a file
- `resolvedLinks` / `unresolvedLinks` — vault-wide link maps

Events: `changed` (file metadata updated), `resolve` (single file resolved),
`resolved` (all files resolved after bulk change).

---

## Dataview Query Layer

Demonstrates vault-as-database. Indexes all files with three metadata sources:
YAML frontmatter, inline fields (`Key:: Value`), and implicit file.* fields.

**DQL output types:** TABLE, LIST, TASK, CALENDAR
**Commands:** FROM (tag/folder/link filter), WHERE, SORT, GROUP BY, FLATTEN, LIMIT
**Pipeline execution:** Commands execute in written order (not SQL-style optimization)

**Implicit fields:** file.name, file.path, file.folder, file.size, file.ctime,
file.mtime, file.tags, file.etags, file.inlinks, file.outlinks, file.aliases,
file.tasks, file.lists, file.day.

**Inline field variants:**
- `Field:: Value` — standalone line, visible
- `[Field:: Value]` — in-sentence, visible
- `(Field:: Value)` — in-sentence, key hidden in reading mode

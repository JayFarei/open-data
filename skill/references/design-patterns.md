# Open Data Persistence Patterns

Reusable architectural patterns for building futureproof persistence systems.
Each pattern includes its structure, when to use it, tradeoffs, and an example.

---

## Pattern 1: File-Per-Entity

**Structure:** Each logical entity (person, project, event, bookmark, etc.) is a
single Markdown file with YAML frontmatter for structured fields and body content
for unstructured/rich content.

```
entities/
├── entity-alpha.md
├── entity-beta.md
└── entity-gamma.md
```

Each file:
```yaml
---
id: entity-alpha
type: project
status: active
created: 2025-01-15
tags:
  - engineering
  - priority
related:
  - "[[entity-beta]]"
---

# Entity Alpha

Free-form content, notes, descriptions, documentation.
Supports full Markdown including links, embeds, code blocks.
```

**When to use:** The default pattern. Use whenever entities have both structured
attributes and unstructured content. Works for notes, bookmarks, contacts, projects,
journal entries, documentation pages, recipes, research papers — anything that
benefits from being both machine-queryable and human-readable.

**Tradeoffs:**
- ✅ Each entity is independently readable and editable
- ✅ Frontmatter provides queryable structured data
- ✅ Body provides unlimited unstructured content
- ✅ Standard Markdown editors can open any file
- ⚠️ Doesn't scale well past ~50,000 files without index optimization
- ⚠️ Bulk operations require scripting over many files
- ⚠️ No built-in referential integrity (links can break)

---

## Pattern 2: Directory-As-Collection

**Structure:** A folder acts as a database table or collection. All files within
share a common schema (enforced by convention, templates, or validation). The
folder name is the collection name.

```
vault/
├── projects/
│   ├── alpha.md          # All share project schema
│   ├── beta.md
│   └── gamma.md
├── people/
│   ├── alice.md          # All share person schema
│   └── bob.md
├── daily-notes/
│   ├── 2025-01-15.md     # Date-based naming convention
│   └── 2025-01-16.md
└── _schemas/             # Optional: schema definitions
    ├── project.md
    └── person.md
```

**When to use:** When you have multiple entity types with distinct schemas. The
folder hierarchy becomes your database schema. Combine with templates to enforce
consistency within each collection.

**Schema enforcement strategies (choose one):**
1. **Template-based:** A template file defines the expected frontmatter. New files
   are created from the template. Enforcement is social/conventional.
2. **Schema file:** A `_schema.md` or `_schema.yaml` file in each collection folder
   defines expected properties and types. Tooling validates against it.
3. **Type property:** Each file includes a `type:` property. Validation rules are
   keyed to the type, regardless of folder location.
4. **Property type registry:** A central `types.json` (Obsidian-style) enforces that
   a given property name always has the same type across all collections.

**Tradeoffs:**
- ✅ Clear organizational hierarchy
- ✅ Folder = collection is intuitive for both humans and tools
- ✅ Easy to apply batch operations per collection (glob patterns)
- ⚠️ Files that belong to multiple collections need a choice: store in one,
  link from others (or use tags instead of folders for cross-cutting concerns)
- ⚠️ Deep nesting (>3 levels) becomes hard to navigate

---

## Pattern 3: Temporal Partitioning

**Structure:** Files are organized by time period, with naming conventions that
encode timestamps. Useful for logs, journals, feeds, changelogs, and any
append-heavy data stream.

```
daily-notes/
├── 2025/
│   ├── 01/
│   │   ├── 2025-01-15.md
│   │   └── 2025-01-16.md
│   └── 02/
│       └── 2025-02-13.md
```

Or flat with ISO-prefixed names:
```
feed/
├── 2025-01-15T10-30-00_bookmark-title.md
├── 2025-01-15T14-22-00_another-item.md
└── 2025-02-13T09-00-00_latest-item.md
```

**Naming conventions for temporal files:**
- `YYYY-MM-DD` for daily granularity (journal, daily notes)
- `YYYY-MM-DDThh-mm-ss` for sub-day granularity (feeds, logs)
- `YYYY-MM-DDThh-mm-ss_slug` for timestamped entities (bookmarks, captures)
- Use hyphens in timestamps (not colons) for filesystem compatibility
- ISO 8601 ordering ensures alphabetical = chronological

**When to use:** Any data that is primarily append-only and naturally ordered by
time. Especially good for ingestion pipelines (RSS feeds, social media captures,
web clippings) where items arrive continuously.

**Tradeoffs:**
- ✅ Natural chronological ordering from file system sorting
- ✅ Easy archival (move old year/month folders)
- ✅ Predictable file naming eliminates collision risk
- ✅ Works well with git (small diffs, clear history)
- ⚠️ Finding a specific item requires knowing its date or searching
- ⚠️ Deeply nested year/month/day can feel over-engineered for small datasets

---

## Pattern 4: Hub-And-Spoke (Index Notes)

**Structure:** A "hub" or "index" note serves as an entry point that links to
related notes via wikilinks. The hub aggregates, curates, and contextualizes
a set of related entities.

```
projects/
├── _index.md              # Hub: links to all projects, status overview
├── alpha.md               # Spoke: individual project
├── beta.md
└── gamma.md
```

Hub file (`_index.md`):
```markdown
---
type: index
scope: projects
---

# Projects

## Active
- [[alpha]] — Main product development
- [[beta]] — Research initiative

## Completed
- [[gamma]] — 2024 Q4 deliverable

## By Priority
1. [[alpha]]
2. [[beta]]
```

**When to use:** When a flat list of files in a folder isn't enough — you need
curated views, ordered lists, narrative context, or multiple groupings of the
same entities. Index notes are the "views" of an open data system.

**Variants:**
- **Map of Content (MOC):** A hub note for a topic area, linking across multiple
  folders. Not tied to folder structure.
- **Table of Contents:** Linear ordering of notes for reading in sequence.
- **Dashboard:** Hub with embedded queries (Dataview) or manually curated metrics.

**Tradeoffs:**
- ✅ Provides human-curated navigation (better than just search)
- ✅ Multiple hub notes can slice the same data differently
- ✅ Hubs are themselves just Markdown files — fully portable
- ⚠️ Hubs must be manually maintained (unless automated with queries)
- ⚠️ Risk of staleness if entities are added/removed without updating hubs

---

## Pattern 5: Typed Links via Frontmatter Properties

**Structure:** Relationships between entities are expressed as named frontmatter
properties containing wikilinks. This creates **typed edges** in the knowledge
graph — not just "A links to B" but "A depends-on B" or "A is-authored-by B."

```yaml
---
title: Project Alpha
depends-on:
  - "[[library-x]]"
  - "[[service-y]]"
owned-by: "[[alice]]"
related:
  - "[[beta]]"
upstream: "[[data-pipeline]]"
---
```

**When to use:** When relationships carry semantic meaning beyond "these two things
are connected." Essential for any domain where the *type* of relationship matters:
dependency graphs, organizational hierarchies, content taxonomies, citation networks.

**Resolution in queries (Dataview example):**
```
TABLE depends-on, owned-by
FROM "projects"
WHERE owned-by = [[alice]]
```

**Tradeoffs:**
- ✅ Relationships are explicitly typed and queryable
- ✅ Frontmatter links are recognized by Obsidian's graph and backlink systems
- ✅ Multiple relationship types per entity
- ⚠️ YAML requires quoting wikilinks: `"[[target]]"`
- ⚠️ Not all tools recognize wikilinks inside YAML values
- ⚠️ Bidirectional relationships must be maintained in both files (or computed)

---

## Pattern 6: Convention-Based Metadata Extraction

**Structure:** Instead of requiring explicit frontmatter for every field, derive
metadata from file names, folder paths, and content patterns. The conventions
themselves become the schema.

**Examples:**
- File name `2025-01-15_meeting-notes.md` → `date: 2025-01-15`, `type: meeting-notes`
- Folder path `projects/alpha/docs/` → `project: alpha`, `category: docs`
- First H1 heading → `title` (if no frontmatter title)
- `#status/active` inline tag → `status: active`

**When to use:** When you want minimal frontmatter overhead but still need
structured queryability. The indexing layer extracts structure from conventions
rather than requiring authors to fill in metadata forms.

**Implementation requirements:**
- Document the naming conventions explicitly (they ARE the schema)
- Build extraction rules into your indexer
- Provide fallback defaults when conventions aren't followed
- Validate convention compliance as a linting step

**Tradeoffs:**
- ✅ Minimal authoring friction — just name the file correctly
- ✅ File names and folder paths are universally portable
- ✅ Works even without any Markdown-aware tooling
- ⚠️ Conventions must be well-documented and consistently followed
- ⚠️ Richer metadata still needs explicit frontmatter
- ⚠️ Extraction rules add complexity to the indexing layer

---

## Pattern 7: Sidecar Metadata Files

**Structure:** When the primary content file can't contain metadata (binary files,
images, PDFs, non-Markdown content), a companion `.md` or `.yaml` file sits
alongside it with the same name.

```
attachments/
├── diagram.png
├── diagram.png.md          # Metadata sidecar for the image
├── report.pdf
└── report.pdf.yaml         # Metadata sidecar for the PDF
```

Sidecar (`diagram.png.md`):
```yaml
---
type: diagram
source: "[[project-alpha]]"
created: 2025-01-15
description: Architecture diagram showing service dependencies
tags:
  - architecture
  - diagrams
---
```

**When to use:** For any non-text assets that need to participate in the knowledge
graph — images, PDFs, videos, datasets, binary files. The sidecar makes them
findable, linkable, and queryable without modifying the original file.

**Naming conventions for sidecars:**
- `filename.ext.md` — Markdown sidecar (most common)
- `filename.ext.yaml` — Pure YAML sidecar (when no prose is needed)
- `filename.ext.meta` — Custom extension (requires tool support)

**Tradeoffs:**
- ✅ Keeps binary files unmodified
- ✅ Sidecars are themselves open-format text files
- ✅ Binary assets gain full queryability and graph participation
- ⚠️ Two files per asset = more files to manage
- ⚠️ File moves must keep sidecars and originals together
- ⚠️ Not all tools recognize sidecar conventions

---

## Pattern 8: Three-Layer Application Data Separation

**Structure:** All application data lives in a single hidden directory (`.<app-name>/`),
strictly separated from user content and organized into three layers based on
deletability and replaceability.

```
vault/
├── .<app-name>/                # All application data in one place
│   ├── config/                 # Layer: Settings & preferences
│   │   ├── app.json            # Core settings
│   │   ├── types.json          # Property type registry
│   │   ├── plugins/            # Plugin/extension settings
│   │   └── workspace.json      # Ephemeral UI state
│   ├── state/                  # Layer: Irreplaceable app state
│   │   ├── kg-links.json       # Manual knowledge graph corrections
│   │   ├── entity-feedback.json # User-verified entity decisions
│   │   └── merge-log.json      # Manual merge/dedup choices
│   └── cache/                  # Layer: Regenerable derived data
│       ├── search-index.json   # Full-text search index
│       ├── link-graph.json     # Resolved link map
│       └── tag-index.json      # Tag → files mapping
├── content/                    # User data (the actual value)
│   ├── projects/
│   └── notes/
└── .gitignore                  # Excludes .<app-name>/cache/
```

**Separation rules:**
1. **User content** — Never touched by app data changes. Survives app uninstall.
2. **App configuration** — Settings, preferences, plugin data. Deletable without
   data loss. Portable between machines.
3. **App state** — Operational data containing irreplaceable user decisions (manual
   link corrections, entity feedback, merge choices). **Not deletable** without
   losing user work. Must be git-tracked.
4. **Cache** — Always regenerable from source files alone. Can be deleted and
   rebuilt. Should be `.gitignore`d.
5. **Frontmatter boundary** — A field belongs in frontmatter (open data) only if
   a different Markdown-based tool would find it meaningful and actionable.
   Otherwise it belongs in the app directory.

**Tradeoffs:**
- ✅ `rm -rf .<app-name>/config/ .<app-name>/cache/` leaves user data and decisions intact
- ✅ Irreplaceable user decisions are protected in `state/`, not mixed into cache
- ✅ Different apps can share the same content directory
- ✅ Git workflows are clean (track config + state, ignore cache)
- ⚠️ Requires discipline to classify data into the correct layer
- ⚠️ Some tools conflate config and content by design
- ⚠️ The state layer is a judgment call: data must contain irreplaceable user decisions to qualify

---

## Pattern Combinations

Most real systems combine multiple patterns:

**Knowledge Management System (like injest.md):**
Patterns 1 + 2 + 3 + 4 + 6 + 8
- File-per-entity for bookmarks/captures (Pattern 1)
- Directory-as-collection for source types: twitter/, github/, reddit/ (Pattern 2)
- Temporal partitioning within collections (Pattern 3)
- Hub notes for curated topic views (Pattern 4)
- Extract metadata from URLs, timestamps, source identifiers (Pattern 6)
- Three-layer app data separation for pipeline state, config, and cache (Pattern 8)

**Project Management / CRM:**
Patterns 1 + 2 + 5 + 4
- File-per-entity for clients, projects, tasks (Pattern 1)
- Directory-as-collection for entity types (Pattern 2)
- Typed links for relationships: owned-by, depends-on, blocks (Pattern 5)
- Dashboard hub notes for status views (Pattern 4)

**Documentation Site / Wiki:**
Patterns 1 + 2 + 4 + 7
- File-per-page (Pattern 1)
- Sections as directories (Pattern 2)
- Navigation via index notes and MOCs (Pattern 4)
- Sidecars for images and embedded assets (Pattern 7)

**Data Ingestion Pipeline:**
Patterns 1 + 3 + 6 + 7 + 8
- File-per-item for ingested content (Pattern 1)
- Temporal organization for feed items (Pattern 3)
- Convention-based metadata from source APIs (Pattern 6)
- Sidecars for non-text attachments (Pattern 7)
- Three-layer separation of pipeline config, state, and cache from content (Pattern 8)

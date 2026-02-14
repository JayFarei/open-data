# Persistence Design Decision Framework

A structured guide for making architectural decisions when designing an open data
persistence layer. Work through these decisions in order — each builds on the previous.

---

## Decision 1: What Are the Entities?

Before choosing any pattern, enumerate what needs to be persisted.

**Questions to answer:**
- What are the primary entity types? (e.g., bookmarks, projects, people, notes)
- Which entities have unstructured content vs. pure structured data?
- How many entities of each type do you expect? (10s? 1000s? 100,000s?)
- Are entities created by humans, automated pipelines, or both?
- What is the lifecycle? (append-only? frequently edited? archived?)

**Decision output:** An entity catalog listing each type, its expected volume,
creation method, and whether it needs a body (unstructured content) or is
metadata-only.

**Rule of thumb:** If an entity has any prose, description, or rich content,
it should be a Markdown file (Pattern 1: File-Per-Entity). If it's purely
structured data with no body, consider whether it should be a row in a
collection-level YAML/JSON file instead of its own file.

---

## Decision 2: How Are Entities Organized?

Choose the primary organizational axis for the file system layout.

**Option A: By entity type (Directory-As-Collection)**
```
vault/
├── projects/
├── people/
├── bookmarks/
└── notes/
```
Best when: Entity types have distinct schemas. Users think in categories.

**Option B: By time (Temporal Partitioning)**
```
vault/
├── 2025/
│   ├── 01/
│   └── 02/
```
Best when: Data is primarily append-only. Chronology is the natural axis.

**Option C: By topic/domain (Flat with Tags)**
```
vault/
├── note-about-rust.md          # tagged: #topic/programming
├── rust-project-plan.md        # tagged: #topic/programming, #type/project
└── meeting-2025-01-15.md       # tagged: #type/meeting
```
Best when: Entities cross-cut categories. Folder hierarchy would force false choices.

**Option D: Hybrid (most common)**
```
vault/
├── projects/           # By type
├── people/             # By type
├── feed/               # By time within type
│   ├── 2025-01-15_bookmark.md
│   └── 2025-01-16_bookmark.md
└── notes/              # Flat with tags
```
Best when: Different entity types have different natural organizing axes.

**Decision criteria matrix:**

| Factor | By Type | By Time | By Topic | Hybrid |
|--------|---------|---------|----------|--------|
| Schema consistency | ✅ High | ⚠️ Mixed | ❌ Low | ✅ Per-section |
| Chronological browsing | ❌ Poor | ✅ Natural | ❌ Poor | ✅ Where needed |
| Cross-cutting queries | ⚠️ Needs tags | ⚠️ Needs tags | ✅ Native | ✅ Flexible |
| Scalability | ✅ Good | ✅ Good | ⚠️ Gets noisy | ✅ Good |
| Simplicity | ✅ Simple | ✅ Simple | ✅ Simple | ⚠️ More complex |

---

## Decision 3: What Is the Metadata Schema?

Define what structured fields each entity type needs.

**Step 1: Identify mandatory properties**
Every entity of a given type MUST have these. They should be enforced.
Common mandatory properties: `type`, `created`, `title` or filename-as-title.

**Step 2: Identify recommended properties**
Most entities SHOULD have these, but missing values are tolerable.
Examples: `tags`, `status`, `source`, `author`.

**Step 3: Identify optional properties**
Only some entities will have these. They extend the schema per-instance.
Examples: `due-date`, `rating`, `priority`, `related`.

**Step 4: Choose property types**
Map each property to a type from the Obsidian-compatible type system:

| Type | YAML representation | Use for |
|------|-------------------|---------|
| `text` | `key: "value"` | Freeform strings, enums, identifiers |
| `number` | `key: 42` | Counts, ratings, priorities, amounts |
| `checkbox` | `key: true` | Boolean flags |
| `date` | `key: 2025-01-15` | Dates without time |
| `datetime` | `key: 2025-01-15T14:30` | Timestamps |
| `list` | `key: [a, b, c]` | Multi-valued text fields |
| `tags` | `tags: [x, y]` | Reserved: hierarchical categorization |
| `aliases` | `aliases: [x, y]` | Reserved: alternative names for linking |
| `link` | `key: "[[target]]"` | Relationships to other entities |
| `link-list` | `key: ["[[a]]", "[[b]]"]` | Multi-valued relationships |

**Step 5: Decide on schema enforcement level**

| Level | Mechanism | When to use |
|-------|-----------|-------------|
| **None** | Conventions only, no validation | Personal notes, low-structure vaults |
| **Template** | Templates pre-fill expected fields | Human-authored content |
| **Type registry** | `types.json` enforces type per property name | Multi-tool environments |
| **Validation** | CI/lint script checks all files against schemas | Pipeline-generated content |
| **Strict** | Files rejected/quarantined if non-conforming | Production data systems |

**Step 6: Apply the Frontmatter Boundary Test**

For each candidate property, ask: **Would a different Markdown-based tool find
this field meaningful and actionable?**

| Answer | Placement | Examples |
|--------|-----------|---------|
| Yes, any Markdown tool can use it | **Frontmatter** (open data) | `tags`, `status`, `source_url`, `starred`, `prev_note: "[[slug]]"` |
| No, requires the app to interpret | **App state** (`.<app-name>/state/`) | KG entity extractions, manual link merge decisions, confidence scores, UUID mappings |
| Derived / regenerable | **Cache** (`.<app-name>/cache/`) | Search indices, computed backlink maps, tag counts |

**Common frontmatter-worthy fields:** `title`, `type`, `created`, `modified`, `tags`, `status`,
`source`, `author`, `aliases`, relationship links (`depends-on`, `owned-by`), `starred`,
`priority`, `due-date`, `rating`.

**Common app-state fields:** Entity extraction results, NLP annotations, manual merge/dedup
decisions, user feedback on AI-generated content, internal UUID mappings, processing pipeline
status, confidence scores.

---

## Decision 4: How Are Relationships Expressed?

Choose how entities reference each other.

**Option A: Inline wikilinks only**
```markdown
This project depends on [[library-x]] and is owned by [[alice]].
```
- Relationships embedded in prose
- Untyped (no semantic distinction between mention and dependency)
- Most natural for human writing

**Option B: Frontmatter typed links only**
```yaml
depends-on:
  - "[[library-x]]"
owned-by: "[[alice]]"
```
- Relationships are explicitly typed and queryable
- Separated from content
- Better for machine processing

**Option C: Both (recommended for most systems)**
```yaml
---
depends-on: ["[[library-x]]"]
owned-by: "[[alice]]"
---
Discussed the dependency on [[library-x]] with [[alice]] today...
```
- Frontmatter for canonical, queryable relationships
- Inline links for contextual mentions
- Index tracks both; queries use frontmatter; graph shows all

**Decision criteria:**

| Factor | Inline only | Frontmatter only | Both |
|--------|------------|-------------------|------|
| Authoring friction | ✅ Low | ⚠️ Medium | ⚠️ Medium |
| Queryability | ❌ Hard to filter by type | ✅ Easy | ✅ Easy |
| Human readability | ✅ Natural | ⚠️ Hidden in YAML | ✅ Best of both |
| Graph richness | ⚠️ Untyped edges | ✅ Typed edges | ✅ Typed + contextual |

---

## Decision 5: How Is the Link Resolution Algorithm Designed?

When a link target is `[[alpha]]`, how does the system find the file?

**Option A: Exact filename match (simplest)**
- Search all `.md` files for one named `alpha.md`
- Case-insensitive matching
- Fail if multiple matches (require disambiguation)

**Option B: Shortest unique path (Obsidian default)**
- If `alpha.md` is unique, `[[alpha]]` suffices
- If duplicates exist, require `[[projects/alpha]]`
- Most concise linking syntax

**Option C: Relative paths**
- Links resolved relative to the linking file's location
- `[[../shared/alpha]]`
- Most predictable, most verbose

**Option D: Absolute vault paths**
- Always from vault root: `[[projects/alpha]]`
- No ambiguity, slightly verbose

**Recommendation:** Start with Option B (shortest path) for human-authored content.
Use Option D (absolute) for machine-generated content. Support both in the resolver.

**Additional resolution features to decide on:**
- [ ] Alias resolution (match against `aliases:` frontmatter values)
- [ ] Case-insensitive matching (recommended: yes)
- [ ] Extension-optional (`.md` can be omitted in links)
- [ ] Automatic link updating on file rename
- [ ] Heading links (`[[file#Heading]]`)
- [ ] Block references (`[[file#^block-id]]`)

---

## Decision 6: What Does the Index Look Like?

The index is the derived data layer that makes files queryable. Define what it tracks.

**Minimum viable index:**
```json
{
  "files": {
    "projects/alpha.md": {
      "frontmatter": { "type": "project", "status": "active" },
      "links": ["library-x.md", "alice.md"],
      "tags": ["engineering", "priority"],
      "headings": ["Overview", "Dependencies", "Timeline"],
      "modified": "2025-01-15T10:30:00Z"
    }
  },
  "resolvedLinks": {
    "projects/alpha.md": { "library-x.md": 2, "people/alice.md": 1 }
  },
  "unresolvedLinks": {
    "projects/alpha.md": { "nonexistent-note": 1 }
  },
  "tags": {
    "engineering": ["projects/alpha.md", "projects/beta.md"],
    "priority": ["projects/alpha.md"]
  }
}
```

**Index storage options:**

| Approach | Persistence | Regeneration | Use case |
|----------|-------------|-------------|----------|
| In-memory only | None — rebuilt on startup | Always fresh | Small vaults, CLI tools |
| JSON file in `.<app-name>/cache/` | File-based | On demand | Medium vaults, simple tooling |
| SQLite in `.<app-name>/cache/` | File-based | On demand | Large vaults, complex queries |
| IndexedDB | Browser storage | On demand | Web/Electron apps (Obsidian's approach) |

**Critical rule:** The index MUST be regenerable from source files alone. It is
a cache, not a data store. If `.<app-name>/cache/` is deleted, the system must rebuild it
without data loss.

---

## Decision 7: How Is Non-Markdown Content Handled?

Define a strategy for binary assets and non-text files.

**Questions:**
- Will the system store images, PDFs, audio, video?
- Do binary assets need metadata (tags, descriptions, relationships)?
- Should assets be co-located with their referencing notes or centralized?

**Option A: Centralized attachments folder**
```
vault/
├── attachments/
│   ├── diagram.png
│   └── report.pdf
├── projects/
│   └── alpha.md          # References ![[diagram.png]]
```

**Option B: Co-located with referencing notes**
```
vault/
├── projects/
│   ├── alpha.md
│   └── alpha-assets/
│       └── diagram.png
```

**Option C: Sidecar metadata (Pattern 7)**
```
vault/
├── attachments/
│   ├── diagram.png
│   └── diagram.png.md    # Metadata sidecar
```

**Recommendation:** Option A for simplicity, Option C when assets need to
participate in the knowledge graph. Option B for self-contained projects
that may be moved or shared independently.

---

## Decision 8: What Are the Git/Sync Considerations?

If the vault will be version-controlled or synced, decide early.

**Should be tracked in git:**
- All user content (`.md` files, assets)
- App configuration (`.<app-name>/config/` minus ephemeral state and secrets)
- App state (`.<app-name>/state/`) — contains irreplaceable user decisions
- Schema definitions (`types.json`, `_schema.md` files)
- Templates

**Should be `.gitignore`d:**
- Cache / derived indices (`.<app-name>/cache/`)
- Workspace/UI state (`.<app-name>/config/workspace.json`)
- Secrets and API keys
- Plugin binaries (if large; track `manifest.json` only)
- OS files (`.DS_Store`, `Thumbs.db`)

**Example `.gitignore`:**
```
.<app-name>/cache/
.<app-name>/config/workspace.json
.<app-name>/config/secrets.*
.DS_Store
Thumbs.db
*.tmp
```

**Sync conflict strategy:**
- Markdown files: Line-level merge works well (git handles it)
- JSON config files: Last-write-wins or manual merge
- Binary assets: No merge possible, keep both versions
- Frontmatter: Most merge-friendly when properties are on separate lines

---

## Decision Summary Template

After working through all decisions, summarize in this format:

```markdown
## Persistence Architecture Decisions

### Entities
- [List entity types with expected volumes]

### Organization
- Primary axis: [type / time / topic / hybrid]
- Directory structure: [describe]

### Schema
- Enforcement level: [none / template / type-registry / validation / strict]
- Mandatory properties: [list]
- Type registry: [yes/no, location]

### Relationships
- Approach: [inline / frontmatter / both]
- Typed link properties: [list]
- Resolution algorithm: [shortest-path / relative / absolute]

### Index / Cache
- Storage: [memory / JSON / SQLite / IndexedDB]
- Location: `.<app-name>/cache/`
- Rebuild trigger: [startup / file-change / manual]

### App State
- Location: `.<app-name>/state/`
- Contents: [list irreplaceable user decisions stored here]
- Git-tracked: Yes

### Assets
- Strategy: [centralized / co-located / sidecar]
- Metadata approach: [frontmatter in sidecar / embedded in referencing note / none]

### Version Control
- Tracked: [list, including `.<app-name>/config/` and `.<app-name>/state/`]
- Ignored: [list, including `.<app-name>/cache/`]
- Conflict strategy: [describe]
```

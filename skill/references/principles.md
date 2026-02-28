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

Application data lives in a single hidden app directory (`.<app-name>/`),
separated from user content and organized into three layers:

| Layer | Contains | Deletable? | Git-tracked? |
|-------|----------|------------|--------------|
| `config/` | Settings, preferences, API keys | Yes (no data loss) | Yes (minus secrets + ephemeral) |
| `state/` | Operational data with irreplaceable user decisions (manual link corrections, entity feedback, merge choices) | **No** (irreplaceable) | Yes |
| `cache/` | Regenerable indices, search caches, computed views | Yes (rebuildable from source files) | No |

**Frontmatter boundary test:** Would a different Markdown-based tool find this
field meaningful and actionable? Yes → frontmatter (open data). No → app directory.

**Test 1:** Can you delete `.<app-name>/config/` and `.<app-name>/cache/` without losing any user data or irreplaceable decisions?

**Test 2:** Is every frontmatter property meaningful outside the app that created it?

**Test 3:** Are irreplaceable user decisions (manual corrections, feedback) in `state/`, not mixed into `cache/` or content files?

## Principle 6: Tool-Agnostic Identity

Files are identified by their path, not by UUIDs, internal IDs, or database keys.
The file name IS the identity. Moving or renaming a file changes its identity —
and the system must handle that explicitly.

**Test:** Does any file reference an opaque ID that has no meaning outside the app?

**Migration hazard: identity-scheme changes.** When renaming files to a new
identity scheme, different layers of the system may still reference the old
names. Content files, app state, and the filesystem can fall out of sync,
creating broken references. Mitigate by updating all references atomically
during migration rather than maintaining a permanent old-to-new mapping layer.

## Principle 7: Composability

The data model must be layerable. Each layer is independently valuable:

- **Layer 0:** Files exist and are readable plain text
- **Layer 1:** YAML frontmatter adds structured metadata
- **Layer 2:** Wikilinks add relationships between files
- **Layer 3:** Indices add queryability (search, graph, backlinks)
- **Layer 4:** Plugins add domain-specific features
- **Layer 5:** Agents add reasoning, chaining, and bulk semantic operations

**Test:** Does each layer add value without requiring the layers above it?

## Principle 8: Apps as Accelerators

The open data layer exists so you can adopt apps without fear of lock-in. An app
that indexes your files, controls a UI, or syncs data remotely is a legitimate
power tool, not a threat. Use it. The principles above guarantee you can walk
away; this principle says you shouldn't have to.

Four things an app can offer that raw files can't:

1. **Index access** — querying the app's index is faster than grep for tasks like finding orphans or running structured queries
2. **UI control** — screenshots, evals, visual inspection, useful for development workflows
3. **Remote services** — sync, publish, and other cloud features that operate on the same open data
4. **Agent surface** — a CLI or API that lets reasoning tools leverage the app's
   index without reimplementing parsing, resolution, or graph traversal

Open data means agents can bypass the app entirely. A good agent surface means
they won't want to. The CLI, headless mode, or API becomes the path of least
resistance, offering the app's speed and capability without requiring its GUI.
This is the strategic complement to portability: make the app worth using, not
just safe to leave.

**Test:** Is the app adding speed or capability on top of data you could still read without it?

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
| **Open data** | Content and metadata meaningful outside the app that created it |
| **App directory** | Hidden `.<app-name>/` directory containing all application data (config, state, cache) |
| **App state** | Operational data with irreplaceable user decisions, stored in `.<app-name>/state/` |
| **Cache** | Derived data structure regenerable from source files, stored in `.<app-name>/cache/` |
| **Frontmatter boundary** | The test: "Would a different Markdown-based tool find this field meaningful?" |
| **Schema enforcement** | Type constraints on properties, applied vault-wide |
| **Link resolution** | Algorithm for turning a link target into a file path |
| **Graceful degradation** | Property of data that remains useful in unsupported tools |
| **Read path** | The route a consumer takes to discover and retrieve data (glob + cat, index query, API call) |
| **Write path** | The route a consumer takes to create or modify data (file creation, API POST, form submit) |
| **Consumer** | Any human, app, agent, or pipeline that reads from or writes to the persistence layer |
| **Agent** | An LLM-powered tool or script that operates on the data via CLI or API |
| **Pipeline** | An automated process that bulk-creates, transforms, or publishes data |
| **Publish pipeline** | A derived-data process that transforms source files into a different output format |
| **Access layer** | The set of interfaces (filesystem, CLI, API, UI) through which consumers interact with persisted data |

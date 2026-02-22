---
name: open-data
description: >
  Architect and co-design futureproof persistence systems built on open data principles.
  Use when designing data layers, choosing storage formats, structuring knowledge bases,
  building file-system-as-database architectures, or evaluating existing systems for
  portability and longevity. Use when user says "design my data model", "how should I
  store this", "is my data portable", "audit my persistence layer", "plan a migration",
  or asks about file-based databases, Markdown schemas, or Obsidian-compatible data formats.
  Do NOT use for general coding tasks, database query optimization, or SQL schema design.
---

# Open Data Architect

A skill for designing persistence systems where user data outlives any particular
application. The governing philosophy: the file system is the database, plain text
is the schema, and conventions replace proprietary formats.

## Instructions

Before producing any output, read the relevant reference files:

- `references/principles.md` — The 7 non-negotiable futureproof principles
- `references/design-patterns.md` — 9 reusable persistence patterns with tradeoffs
- `references/decision-framework.md` — Step-by-step architectural decision guide
- `references/obsidian-spec.md` — Obsidian data model technical reference

Then choose the appropriate workflow below based on what the user needs.

### Step 1: Understand the Request

Determine what the user is asking for:

- **Designing a new system?** → Go to Step 2A
- **Evaluating an existing system?** → Go to Step 2B
- **Thinking through a specific tradeoff?** → Go to Step 2C

Ask clarifying questions if the domain, entity types, or constraints are unclear.
Do not assume — the user's domain matters enormously.

### Step 2A: Design a New Persistence Layer

1. Read `references/design-patterns.md` and `references/decision-framework.md`
2. Ask the user about entity types, expected volumes, creation method, and lifecycle
3. Walk through the 8 decisions in `references/decision-framework.md` with the user
4. Propose a directory structure, naming conventions, and metadata schema
5. Produce a persistence specification using the template in `references/persistence-spec.md`
6. Validate every decision against the principles in `references/principles.md`

Expected output: A complete persistence specification document.

### Step 2B: Audit an Existing System

1. Read `references/auditor-rubric.md` for the 60-point scoring criteria
2. Ask the user to describe their current persistence approach
3. Score each of the 20 criteria (6 categories) from 0-3
4. Identify lock-in points, proprietary dependencies, and portability risks
5. Produce a scored audit report with specific remediation recommendations
6. If migration is needed, propose incremental steps

Expected output: An audit report with scores, findings, and remediation plan.

### Step 2C: Co-Design a Specific Decision

1. Read the relevant reference files for the decision area
2. Propose 2-3 architectural options with explicit tradeoff analysis
3. Stress-test each option against the principles in `references/principles.md`
4. Help the user arrive at a decision with clear rationale
5. Document the decision as an Architecture Decision Record:
   Context → Decision → Consequences → Alternatives Considered

Expected output: An ADR documenting the decision and reasoning.

### Step 3: Validate Output

Before finalizing any deliverable, verify:

- [ ] Every design decision can answer: "If the app disappears, is the data still usable?"
- [ ] No proprietary binary formats are required for core functionality
- [ ] Derived data (indices, caches) is regenerable and lives in `cache/`
- [ ] App data is separated into `config/`, `state/`, and `cache/` within `.<app-name>/`
- [ ] Each frontmatter field passes the boundary test ("Would a different Markdown tool find this meaningful?")
- [ ] Non-regenerable app state (user decisions, manual corrections) is in `state/`, not mixed into `cache/` or content
- [ ] File paths serve as identifiers, not UUIDs or database keys
- [ ] Non-standard syntax degrades gracefully to readable text in other tools

## Examples

### Example 1: Designing a Knowledge Management System

User says: "I'm building a tool that ingests bookmarks from Twitter, GitHub stars,
and Reddit saves into LLM-generated summaries. How should I structure the data?"

Actions:
1. Read design-patterns.md — identify Patterns 1, 2, 3, 6, 8 as applicable
2. Walk through entity modeling: bookmark entities with source metadata
3. Propose hybrid directory structure: by source type + temporal partitioning
4. Define frontmatter schema with mandatory fields (type, source, created, url)
5. Produce persistence specification

Result: Complete spec with directory layout, naming conventions, YAML schema,
and index strategy — all in plain Markdown files.

### Example 2: Auditing a Notion-Based System

User says: "My team uses Notion for everything. How locked in are we?"

Actions:
1. Read auditor-rubric.md
2. Score Notion against 20 criteria (typical result: ~18/60, F rating)
3. Identify critical lock-in: cloud-only, proprietary block model, limited export
4. Propose migration path: Notion export → Markdown conversion → open vault

Result: Audit report with scores and a step-by-step migration plan.

### Example 3: Choosing a Link Resolution Strategy

User says: "Should I use relative paths or shortest-path for internal links?"

Actions:
1. Read obsidian-spec.md section on link resolution
2. Present 3 options with tradeoff matrix (shortest path, relative, absolute)
3. Recommend based on user's context (human-authored vs. machine-generated)
4. Document as an ADR

Result: Architecture Decision Record with clear rationale.

## Troubleshooting

### Output is too abstract
Cause: Not enough domain context from the user.
Solution: Ask specific questions — entity types, volumes, creation method,
query patterns, team size. The decision framework requires concrete answers.

### Principles conflict with user constraints
Cause: Real-world systems sometimes need pragmatic tradeoffs.
Solution: Document the deviation explicitly. State which principle is being
relaxed, why, and what the migration cost would be to fix it later.

### User wants code, not architecture
Cause: This skill produces specifications, not implementations.
Solution: Produce the spec first, then hand off to appropriate coding tools.
The spec serves as the requirements document for implementation.

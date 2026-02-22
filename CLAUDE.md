# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

This repo is a **Claude Code skill** (installed via `npx skills add JayFarei/open-data`), not a traditional software project. There is no build system, no tests, no dependencies. The deliverable is a set of Markdown files that Claude Code loads as context when the skill is triggered.

## Repository Structure

```
skill/
├── SKILL.md                          # Main skill definition (entry point)
└── references/
    ├── principles.md                 # 7 non-negotiable futureproof persistence principles
    ├── design-patterns.md            # 9 reusable persistence patterns
    ├── decision-framework.md         # 8-step architectural decision guide
    ├── auditor-rubric.md             # 60-point scoring rubric (20 criteria, 6 categories)
    ├── obsidian-spec.md              # Obsidian data model technical reference
    └── persistence-spec.md           # Persistence specification template
```

## How the Skill Works

`SKILL.md` is the entry point. Its YAML frontmatter defines trigger phrases. The body contains three workflows:

- **Step 2A (Design):** Walks users through entity modeling, directory structure, naming conventions, and produces a persistence specification using `persistence-spec.md` as template
- **Step 2B (Audit):** Scores existing systems against `auditor-rubric.md` (0-3 per criterion, 60 points max, letter grades A-F)
- **Step 2C (Co-Design):** Compares architectural options for a specific decision, stress-tests against principles, outputs an ADR

Each workflow instructs Claude to read specific reference files before producing output. All outputs are validated against `principles.md`.

## Key Conventions

- The 7 principles in `principles.md` are **non-negotiable**. Every design recommendation must pass the tests listed there (e.g., "Can you `ls` and `cat` your way to understanding the data?").
- Use the **Design Vocabulary** table at the bottom of `principles.md` for consistent terminology (vault, note, frontmatter, property, wikilink, backlink, etc.).
- The audit rubric uses a strict 0-3 scoring scale per criterion. Scores map to letter grades: 51-60 = A, 41-50 = B, 31-40 = C, 21-30 = D, 0-20 = F.
- Pattern numbers (Pattern 1-9) in `design-patterns.md` are referenced by number throughout the skill. Don't renumber them.

## Editing Guidelines

- Reference files are loaded into context at runtime, so keep them concise. Avoid verbose explanations when a table or code block suffices.
- `persistence-spec.md` is a fill-in-the-blanks template. Placeholder text in `[brackets]` is meant to be replaced by the user's domain-specific content.
- Cross-references between files use plain English ("see `references/principles.md`"), not wikilinks or relative Markdown links.

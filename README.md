# open-data

The software stack is changing so fast that one of the key principles I follow for building something durable is to rely on an open data model. The logic layer is disposable. Models get replaced, frameworks get rewritten, orchestration patterns come and go. Your data doesn't have to.

Keep the data model in plain text and build intelligence through indexing. Files in folders. Markdown for content. YAML for structure. Naming conventions for organization. The smart stuff (search indices, link graphs, computed views) sits in a disposable layer on top. Delete it, rebuild it, replace it with something better. The files underneath don't care.

Your application becomes a view layer over data that exists before it and will exist after it. When a better model ships, it reads the same files. When a better framework appears, it points at the same folder. The data never migrates. The application stays simple because it doesn't try to own the data. It just presents it.

## Install

```bash
npx skills add JayFarei/open-data
```

## What it does

A skill for designing persistence systems where user data outlives any particular application. Three workflows:

- **Design** a new persistence layer: walk through entity modeling, directory structure, naming conventions, metadata schemas, and produce a complete persistence specification
- **Audit** an existing system: score it against a 60-point rubric across 6 categories (data format, file system independence, relationship portability, configuration separation, migration viability, extension safety)
- **Co-design** a specific decision: compare architectural options with explicit tradeoff analysis, stress-test against open data principles, and document as an Architecture Decision Record

## What's included

| File | Description |
|------|-------------|
| `skill/SKILL.md` | Main skill definition with workflows, examples, and validation checklist |
| `skill/references/principles.md` | 7 non-negotiable futureproof persistence principles |
| `skill/references/auditor-rubric.md` | 60-point scoring system across 20 criteria |
| `skill/references/decision-framework.md` | 8-step architectural decision guide |
| `skill/references/design-patterns.md` | 9 reusable persistence patterns with tradeoffs |
| `skill/references/obsidian-spec.md` | Obsidian data model technical reference |
| `skill/references/persistence-spec.md` | Persistence specification template |

## License

MIT

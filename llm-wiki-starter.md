# LLM Wiki Starter

A personal knowledge base that builds and maintains itself with an LLM agent (Claude Code, Codex, or any agent with file-system access).

Based on [Karpathy's llm-wiki methodology](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), with production-tested extensions.

---

## How to Use This File

1. Copy this file to your vault root and rename it `CLAUDE.md`
2. Create `purpose.md` in the same directory (see template at the bottom)
3. Create the directory structure below
4. Tell your agent: **"处理 inbox"** / **"process inbox"** to start

---

## Directory Layout

```
vault/
├── CLAUDE.md          ← this file (schema + rules)
├── purpose.md         ← why this wiki exists (customize this)
├── raw/               ← source documents, NEVER modified by agent
│   ├── inbox/         ← DROP ZONE: new files land here, processed then moved out
│   ├── assets/        ← downloaded images and attachments
│   └── <Discipline>/  ← one subfolder per subject area (auto-created on confirmation)
└── wiki/
    ├── index.md       ← master catalog of all wiki pages (agent's navigation hub)
    ├── log.md         ← append-only activity log
    ├── overview.md    ← high-level synthesis of the whole knowledge base
    └── <pages>/       ← one .md file per entity, concept, idea, or topic
```

---

## Wiki Page Types

| Type | Naming | Purpose |
|---|---|---|
| **idea** | `idea-<slug>.md` | A specific idea, creative concept, or shower thought |
| **business** | `biz-<slug>.md` | Business model, opportunity, market, company |
| **research** | `research-<slug>.md` | Research topic, paper summary, finding |
| **project** | `project-<slug>.md` | An ongoing or planned personal project |
| **person** | `person-<slug>.md` | A person, author, thinker, contact |
| **concept** | `concept-<slug>.md` | An abstract concept, framework, mental model |
| **resource** | `resource-<slug>.md` | A tool, book, article, dataset, or reference |
| **topic** | `topic-<slug>.md` | A broad subject area that links many other pages |

Pages can freely link to each other using `[[wiki/page-name]]` Obsidian-style wikilinks.

---

## Frontmatter Standard

Every wiki page must begin with YAML frontmatter:

```yaml
---
title: Human-readable title
type: idea | business | research | project | person | concept | resource | topic
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: 0
---
```

For **research** pages derived from a local file, add `source_pdf`:

```yaml
---
title: Paper title
type: research
tags: [tag1, tag2]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: 1
source_pdf: "[[raw/<Discipline>/exact-filename.pdf]]"
---
```

- `source_pdf` must use `[[wikilink]]` format so it is clickable in Obsidian.
- The path must be the exact relative path from vault root, **after** the file has been moved to its discipline folder.

---

## Inbox Workflow

**Trigger**: user says anything equivalent to "处理 inbox" / "process inbox".

### Step 1 — Analyze (before writing anything)

Read each file in `raw/inbox/` and output a brief analysis:
- What type of knowledge does it contain?
- Which existing wiki pages does it relate to?
- Any contradictions with existing knowledge?
- Suggested discipline folder

Do not write any files yet. Present the analysis and wait if any new discipline folder needs to be confirmed (see rule below).

### Step 2 — Execute

For each file:

1. **Classify** → map to a `raw/<Discipline>/` subfolder
   - Use existing subfolders when they match
   - If no existing folder fits: pause and ask the user to confirm a new folder name before creating it
   - Continue processing other files that fit existing folders without waiting

2. **Check for duplicates** (FATAL-006):
   - Grep `wiki/index.md` for author + year + title keywords
   - Grep all wiki frontmatter for the exact file path
   - If a match exists: UPDATE that page, do NOT create a new one

3. **Move** the file from `raw/inbox/` to `raw/<Discipline>/filename`

4. **Create** `wiki/research-<slug>.md` (or appropriate type):
   - Full content: core findings, methods, key data, limitations, cross-links
   - `source_pdf: "[[raw/<Discipline>/filename]]"` — path after the move

5. **Update** relevant `topic-` pages

6. **Update** `wiki/topic-academic-methodology.md` if the paper introduces a new method, protocol, or analytical technique (see [Academic Toolbox](#academic-toolbox) below). Add a row to the matching domain section; create a new section if no suitable one exists.

7. **Update** `wiki/index.md` and append to `wiki/log.md`

8. **Update** `wiki/overview.md` — always update after every ingest, even briefly

9. **Report**: how many files processed, where filed, pages created/updated

**Inbox is cleared after each run** — every file is moved to its discipline folder.

---

## Ingest Workflow

**Trigger**: user provides a source (URL content, pasted text, image, file path).

Same two-step pattern:

1. **Analyze first**: identify entities, concepts, connections to existing wiki, contradictions
2. **Then generate**: create/update wiki pages, update index + log + overview
3. **Update toolbox**: if the source is a **file (PDF / document)** and introduces a new method or analytical technique, add a row to `wiki/topic-academic-methodology.md` (URLs, pasted text, and images do not trigger this step)

Do not ask for discussion before processing. Process fully, then report.

---

## Query Workflow

When the user asks a question:

1. Read `wiki/index.md` to identify relevant pages
2. Read those pages in full
3. Synthesize an answer with `[[wikilinks]]` to sources
4. If the answer is substantive and reusable, save it as a new wiki page and update index + log

---

## Lint Workflow

When asked to lint or health-check the wiki:

1. Scan all pages in `wiki/` for:
   - Broken wikilinks
   - Orphan pages (no inbound links)
   - Missing `source_pdf` on research pages
   - Concepts mentioned without their own page
   - Pages with no outbound links
2. Report findings as a prioritized list
3. Fix obvious issues automatically
4. Flag deeper contradictions or gaps for the user to decide on

---

## Academic Toolbox

A standing page at `wiki/topic-academic-methodology.md` that serves as a **method index**: given a research question, find the right tool.

### Purpose

Every time a paper with a methodological contribution is ingested, its method is added here as a row. Over time this page becomes the wiki's "what can I measure / how do I test X" lookup, more useful than searching individual research pages.

### Page structure

The page is organized into domain sections. Each section is a lookup table:

```markdown
### [Domain Name]

| 我要... | 候选方法 | 第一选 |
|---|---|---|
| [research question in plain language] | [method name + key idea] | [[research-slug]] |
```

- **我要...** — the research question or goal, phrased as an action ("measure X", "test whether Y", "compare Z")
- **候选方法** — the method(s) that apply, with a one-line description of the key idea
- **第一选** — wikilink to the source research page that introduced or best exemplifies this method

### When to update

- **Inbox workflow**: after creating the research page, check if the paper introduces a new method → add a row to the matching domain section
- **Ingest workflow**: same rule applies to any ingested source
- **New domain**: if no existing section fits, create a new `### [Domain]` heading
- **Increment `sources:`** in the toolbox frontmatter each time a new row is added

### Frontmatter template

```yaml
---
title: 学术方法学 / Academic Methodology 分类与索引
type: topic
tags: [方法学, 索引, 分类, academic, methodology]
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources: 0   # incremented each time a new method row is added
---
```

---

## Discipline Folders

Add rows as new folders are confirmed by the user:

| Folder | Scope |
|---|---|
| *(add rows here as confirmed)* | |

---

## Language

- Wiki page content: match the user's language (default to the language they write in)
- Frontmatter field keys: English
- This schema file: English

---

## Style Guidelines

- Pages should be dense and useful, not padded. No fluff.
- Every non-trivial claim should note its source (inline or in a `## Sources` section).
- Prefer many specific pages over few vague ones. Granularity compounds well.
- Cross-link aggressively. A page with no outbound links is a missed opportunity.
- `overview.md` is the one place for synthesis across everything — keep it current after every ingest.

---

## FATAL Rules

These override all other instructions. No exceptions.

- **FATAL-001** — Any bulk file move or directory restructure: state the plan first, wait for user confirmation, then execute.
- **FATAL-002** — `wiki/` content is never deleted, only archived. `raw/` files are never modified or deleted. Exception: files in `raw/inbox/` may be moved to their discipline folder during Inbox Workflow — this is the only permitted file-system operation on `raw/`.
- **FATAL-003** — Before renaming or moving any file, find and update every `[[wikilink]]` pointing to it.
- **FATAL-004** — Operations touching 3+ files follow PLAN → CONFIRM → EXECUTE. No silent batch edits.
- **FATAL-005** — Every `research-` page derived from a file in `raw/` must contain `source_pdf:` in frontmatter with the exact relative path. A research page without `source_pdf` is incomplete.
- **FATAL-006** — Before creating any new `research-` or `resource-` page, search for duplicates: (a) Grep `wiki/index.md` for author + year + title keywords; (b) Grep all wiki frontmatter for the exact source path. If a match exists, UPDATE that page instead of creating a new one.

---

## Lazy Loading

- Always start with `wiki/index.md`. Only read specific pages when needed.
- Use the `limit` parameter when reading large files. Fetch in chunks, not all at once.

---

---

# purpose.md Template

*Copy the content below into a separate `purpose.md` file in your vault root. Fill in your own details.*

```markdown
# Purpose

This knowledge base serves [describe your identity / role].

---

## Domain 1: [Name, e.g. Research / Academic]

**Goal**: [What you want to achieve with this domain]

**Scope**: [What types of content belong here]

**Core uses**:
- [Use case 1]
- [Use case 2]

---

## Domain 2: [Name, e.g. Writing / Creative / Professional]

**Goal**: [What you want to achieve with this domain]

**Scope**: [What types of content belong here]

**Core uses**:
- [Use case 1]
- [Use case 2]

---

## Cross-Domain Connections

[Optional: describe recurring patterns that connect your domains]

---

## How Claude Should Use This File

- **On ingest**: determine which domain new content belongs to; check for cross-domain resonance
- **On query**: draw from the relevant domain; flag cross-domain insights explicitly
- **On lint**: assess each domain's health separately
```

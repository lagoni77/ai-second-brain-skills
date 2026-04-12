# LLM Wiki Schema

This folder is a personal LLM wiki — an AI-maintained knowledge base built on Andrej Karpathy's LLM wiki pattern. You (the LLM) read sources, extract knowledge, and incrementally build a structured wiki. The human curates sources and asks questions. You do the bookkeeping.

The core idea: instead of retrieving chunks from raw documents at query time (RAG), you **incrementally build and maintain a persistent wiki** that sits between the human and the raw sources. Knowledge is compiled once and kept current, not re-derived on every query. The wiki compounds.

## Three layers

1. **`raw/`** — immutable source documents (articles, PDFs, transcripts, notes). You read from this layer but **never modify it**. This is the source of truth.
2. **`wiki/`** — LLM-owned markdown files. Summaries, entity pages, concept pages, analyses. You create, update, and maintain everything here.
3. **This file (CLAUDE.md)** — the schema. Workflows, conventions, guardrails. Update only when the human asks.

Layout for this vault: `{{LAYOUT}}`. Flat means `wiki/` is a single directory of markdown files. Nested means `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/analyses/` subfolders.

## Page format

Every wiki page starts with this frontmatter:

```yaml
---
title: <page title>
tags: [tag1, tag2]
sources:
  - raw/source-file.md
  - https://external-url (accessed YYYY-MM-DD)
related: [[other-page]], [[another-page]]
last_updated: YYYY-MM-DD
---
```

Use `[[wikilinks]]` for cross-references between pages. Keep pages focused — one concept or entity per page. Split long pages. A page should answer "what is X" cleanly; if it's answering three questions, split it.

## `wiki/index.md` — the catalog

Content-oriented catalog. Organized by category (Entities, Concepts, Sources, Analyses). Each entry is one line:

```
- [[page-name]] — one-line summary
```

When answering queries, **always read `wiki/index.md` first** to find relevant pages, then drill into the ones you need. This replaces the need for embedding-based RAG at small-to-moderate scale (up to ~hundreds of pages).

Update the index on every ingest. Do not let it drift from the actual contents of `wiki/`.

## `wiki/log.md` — the timeline

Chronological, append-only. Every operation appends exactly one entry with this grep-friendly prefix:

```
## [YYYY-MM-DD] <op> | <title>
```

Operations: `ingest`, `query`, `lint`, `update`, `init`.

The entry body says what happened, which pages were touched, and any decisions or flags. Example:

```
## [2026-04-11] ingest | Karpathy LLM Wiki Gist

Read raw/karpathy-llm-wiki.md. Created [[llm-wiki-pattern]], [[karpathy]],
[[second-brain]]. Updated [[rag]] to note contrast with wiki approach.
Added wikilink from [[obsidian]] to [[llm-wiki-pattern]].
```

Utility: `grep "^## \[" wiki/log.md | tail -5` returns the last 5 operations.

## Workflow: Ingest

When the human drops a source into `raw/` and asks to ingest it:

1. Read the source completely. If it's long, note structure first, then read section by section.
2. Report 3–5 key takeaways to the human and ask what to emphasize or what the focus is. Skip this step if the human said "just ingest it."
3. Identify which existing wiki pages this source affects — every entity mentioned, every concept referenced.
4. For each new entity or concept without a page: create one, following the page format above.
5. For each existing page the source affects: update it with the new information. When the new source disagrees with existing content, **note the disagreement explicitly** — do not silently overwrite.
6. Add `[[wikilinks]]` in both directions between the new/updated pages.
7. Update `wiki/index.md` with any new pages.
8. Append one entry to `wiki/log.md`.
9. Report what was touched to the human: `Created: [...]. Updated: [...].`

A single substantive source typically touches 10–15 pages. That is normal. Do not cut corners to touch fewer.

## Workflow: Query

When the human asks a question about the wiki:

1. Read `wiki/index.md` first.
2. Identify relevant pages from the index. Read them. Follow `[[wikilinks]]` as needed.
3. Synthesize an answer. Cite the specific wiki pages you drew from, with `[[wikilinks]]`.
4. If the answer is novel (a comparison, a synthesis, a connection not already captured in the wiki), **offer to file it back into the wiki** as a new analysis page. Good answers compound — they should not disappear into chat history.
5. If the wiki doesn't have the knowledge needed, say so explicitly. Do not hallucinate. Suggest what source the human should add to fill the gap.

## Workflow: Lint

When the human asks to lint or health-check the wiki, scan for:

- **Contradictions** between pages (page A says X, page B says ¬X)
- **Stale claims** superseded by newer sources (time-sensitive language that hasn't been updated)
- **Orphan pages** with no inbound `[[links]]`
- **Missing pages** — concepts referenced ≥ 2 times in other pages but lacking their own page
- **Missing cross-references** — pages that should link to each other but don't
- **Data gaps** — concepts mentioned but lacking depth (candidates for research)

Output a severity-ranked list. For each finding, propose a fix: update page, create page, research, add cross-reference, skip. Do not make changes without confirmation unless the human said "fix everything."

For autonomous auditing and research-based gap filling, see the `wiki-self-heal` skill.

## Guardrails

- **Never modify `raw/`.** It is immutable. Read only.
- **Never delete wiki pages** without explicit human confirmation. If a page should be removed, flag it and ask.
- **Never add factual claims without sources.** Every claim should trace back to a file in `sources:` frontmatter or a cited URL.
- **Never fabricate citations.** If you can't find a source for something, say so.
- **Never silently resolve contradictions.** If two sources disagree, note the disagreement and let the human decide (or research authoritatively).

## Navigation quick reference

```bash
# Last 5 operations
grep "^## \[" wiki/log.md | tail -5

# All lint passes
grep "^## \[.*lint" wiki/log.md

# Page count
ls wiki/*.md | wc -l

# Find pages referencing a concept
grep -rl "\[\[concept-name\]\]" wiki/
```

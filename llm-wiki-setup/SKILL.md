---
name: llm-wiki-setup
description: Initialize a Karpathy-style LLM wiki (personal knowledge base / AI second brain) in a folder. Creates the CLAUDE.md schema, AGENTS.md mirror, raw/wiki directory structure, and index.md/log.md scaffolding — a complete working vault in one pass. Use when the user asks to "set up an LLM wiki", "create a second brain", "build a personal knowledge base", "install the Karpathy wiki pattern", "initialize a knowledge vault", or points at an empty folder and wants to turn it into an AI-maintained wiki.
---

# llm-wiki-setup

Install the Karpathy LLM wiki pattern into a folder. Produces a working vault a beginner can use on the first try.

## What gets created

```
<vault>/
├── CLAUDE.md              # full schema — workflows, conventions, guardrails
├── AGENTS.md              # minimal mirror for Codex and other agents
├── .gitignore
├── raw/                   # immutable source documents (LLM reads, never writes)
│   └── .gitkeep
└── wiki/                  # LLM-owned markdown (LLM creates and maintains)
    ├── index.md           # catalog of every page, organized by category
    └── log.md             # chronological operation log with grep-friendly prefix
```

Optional additions the user may request:

- `wiki/hot.md` — rolling 500-char buffer for fast recent-context lookup
- Nested wiki subfolders: `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/analyses/`

## Setup flow

Ask these three questions in a single message:

1. **Vault path?** Absolute path. Will be created if it doesn't exist.
2. **Flat or nested wiki?** Flat is the default and recommended for most cases (Karpathy's preference). Nested adds subfolders for larger or mixed-topic vaults.
3. **Include hot cache?** Optional `wiki/hot.md` rolling buffer for executive-assistant use cases. Default: no.

## Build steps

Once the user answers:

1. Create the vault directory if missing: `mkdir -p <vault>`
2. Create `raw/` and `wiki/` subdirectories.
3. If nested: also create `wiki/entities/`, `wiki/concepts/`, `wiki/sources/`, `wiki/analyses/`.
4. Copy `templates/CLAUDE.md` to `<vault>/CLAUDE.md`. Replace the `{{LAYOUT}}` placeholder with `flat` or `nested`.
5. Copy `templates/AGENTS.md` to `<vault>/AGENTS.md`.
6. Copy `templates/index.md` to `<vault>/wiki/index.md`.
7. Copy `templates/log.md` to `<vault>/wiki/log.md`. Replace `{{DATE}}` with today's date (YYYY-MM-DD).
8. Copy `templates/gitignore` to `<vault>/.gitignore`.
9. Create `<vault>/raw/.gitkeep` (empty file).
10. If hot cache requested: create `<vault>/wiki/hot.md` with a header comment only.
11. If `<vault>` is not already a git repo: `git init && git add . && git commit -m "chore: initialize LLM wiki"`.

## Report to user

On completion, print:

```
LLM wiki initialized at <vault>

Layout: <flat|nested>
Hot cache: <yes|no>

Next steps:
1. Drop a source into <vault>/raw/ (PDF, markdown, web clip, transcript)
2. Open Claude Code in <vault>
3. Say: "Ingest the new source I just added to raw/"

The LLM will read it, build the wiki, and update the index.
```

## Notes

- The CLAUDE.md template is the hero file — it encodes the three-layer architecture, page format, workflows (ingest/query/lint), and guardrails. Keep it intact during the copy; only the `{{LAYOUT}}` placeholder is substituted.
- The AGENTS.md template is a minimal mirror so OpenAI Codex and other non-Claude agents can also operate the vault.
- Do not overwrite existing files without explicit user confirmation. If any target file already exists, stop and ask.
- The vault is just a git repo of markdown files — no database, no embeddings, no vector store. That is intentional.

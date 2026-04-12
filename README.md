# AI Second Brain Skills

Two Claude Code skills that turn any folder into a Karpathy-style LLM wiki — a persistent, AI-maintained knowledge base that compounds over time.

Based on Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) and inspired in structure by his [autoresearch](https://github.com/karpathy/autoresearch) loop.

## What's in the box

**`llm-wiki-setup`** — One-shot installer. Scaffolds a complete working vault: `CLAUDE.md` schema, `AGENTS.md` mirror, `raw/` and `wiki/` folders, `index.md` catalog, and `log.md` timeline. Three questions, thirty seconds, working second brain.

**`wiki-self-heal`** — Autonomous audit + research-fill pass. Scans the wiki for contradictions, stale claims, orphan pages, missing pages, missing cross-references, and data gaps. Researches high-priority gaps with quality-gated web research (≥2 independent sources per claim, SEO spam rejected). Commits changes to a dedicated branch. Never auto-merges — you review.

Together they give you: a working wiki + a loop that keeps it learning while you sleep.

## Requirements

- [Claude Code](https://claude.com/claude-code) CLI installed and authenticated
- `git` (for wiki history + self-heal branches)
- Optional: [Obsidian](https://obsidian.md) — not required, but makes the vault pretty. The wiki is just markdown files; any editor works.
- Optional for `wiki-self-heal`: Apify, Firecrawl, or Exa MCPs for higher-quality research (defaults to WebSearch + WebFetch)

## Install

Claude Code skills live in `~/.claude/skills/`. Clone this repo and symlink or copy both skill folders into place:

```bash
git clone https://github.com/NulightJens/ai-second-brain-skills.git
cd ai-second-brain-skills

# Symlink (recommended — you get updates via git pull)
ln -s "$(pwd)/llm-wiki-setup"  ~/.claude/skills/llm-wiki-setup
ln -s "$(pwd)/wiki-self-heal"  ~/.claude/skills/wiki-self-heal

# Or copy (independent copies)
# cp -r llm-wiki-setup  ~/.claude/skills/
# cp -r wiki-self-heal  ~/.claude/skills/
```

Restart Claude Code. Both skills will be auto-discovered.

## Usage

### Step 1 — Set up a vault

In Claude Code, from anywhere:

> "Set up an LLM wiki at `~/my-second-brain`"

The `llm-wiki-setup` skill will ask three questions (path, flat or nested layout, hot cache yes/no), then scaffold the vault and initialize git.

### Step 2 — Ingest your first source

Drop a PDF, web clip, transcript, or markdown file into `<vault>/raw/`. Then open Claude Code inside the vault:

```bash
cd ~/my-second-brain
claude
```

And say:

> "Ingest the new source I just added to `raw/`"

Claude will read the source, extract concepts and entities, create or update wiki pages, update `wiki/index.md`, and append to `wiki/log.md`. A single substantive source typically touches 10–15 pages.

### Step 3 — Query it

From inside the vault:

> "Based on the wiki, what are the three biggest points of disagreement across my sources on `<topic>`?"
> "Where are the gaps in my wiki right now?"
> "Write a comparison page for X vs Y and file it back into the wiki."

Good answers compound — they get filed back as new pages.

### Step 4 — Let it learn on repeat

On demand:

> "Run wiki-self-heal on this vault"

Or on a schedule — see `wiki-self-heal/references/scheduling.md` for launchd/cron, GitHub Actions, and Claude Code trigger recipes.

The self-heal skill will:
1. Audit the wiki for the 6 gap types
2. Pick the top 3 high-severity, skill-addressable gaps
3. Research each with quality gates (≥2 independent sources, recent, non-spam)
4. Write findings into new or updated wiki pages with inline citations
5. Commit everything to a `wiki-heal/<date>` branch
6. Report — you review with `git diff main..wiki-heal/<date>` and merge if satisfied

**Never schedule a skill you haven't run manually first.**

## Architecture at a glance

```
<your-vault>/
├── CLAUDE.md              # schema — workflows, conventions, guardrails
├── AGENTS.md              # minimal mirror for Codex / non-Claude agents
├── raw/                   # immutable source documents (read-only)
└── wiki/                  # LLM-owned markdown (created and maintained by the LLM)
    ├── index.md           # catalog of every page
    ├── log.md             # chronological operation log
    └── <pages>.md         # entity / concept / analysis pages
```

Three operations: **ingest** (add sources), **query** (ask questions), **lint** (self-heal). All documented in the `CLAUDE.md` the setup skill installs into your vault.

## Why this over RAG

Traditional RAG retrieves chunks at query time — the LLM rediscovers knowledge from scratch on every question. Nothing accumulates.

The wiki pattern is different: the LLM **incrementally builds and maintains** a persistent, interlinked set of markdown files. When you add a new source, it doesn't just index it — it reads it, extracts the knowledge, integrates it into existing pages, flags contradictions, strengthens the evolving synthesis. Knowledge compiles once and is kept current.

At small-to-moderate scale (up to hundreds of pages) it beats vector-DB RAG on token efficiency, simplicity, and transparency. No embeddings, no vector store, no chunking pipeline — just markdown and an LLM that knows the conventions.

For large-scale (millions of documents) or many-user enterprise search, traditional RAG still wins. This is a personal-to-team tool.

## Credits

- Pattern and gist by [Andrej Karpathy](https://github.com/karpathy) — [original gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Loop structure and autonomy directives inspired by Karpathy's [autoresearch](https://github.com/karpathy/autoresearch)
- Skill packaging by [@NulightJens](https://github.com/NulightJens)

## License

MIT

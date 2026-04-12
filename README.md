# AI Second Brain Skills

> Your folder becomes your app. No database. No embeddings. Just markdown — and an AI that knows where to look.

Two Claude Code skills that turn any folder into a compounding, AI-maintained knowledge base. Based on Andrej Karpathy's [LLM wiki pattern](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f), with a research loop inspired by his [autoresearch](https://github.com/karpathy/autoresearch) project.

## TL;DR

```bash
git clone https://github.com/NulightJens/ai-second-brain-skills.git ~/ai-second-brain-skills
ln -s ~/ai-second-brain-skills/llm-wiki-setup  ~/.claude/skills/llm-wiki-setup
ln -s ~/ai-second-brain-skills/wiki-self-heal  ~/.claude/skills/wiki-self-heal
```

Then in Claude Code:

1. `"Set up an LLM wiki at ~/my-second-brain"`
2. Drop a source into `raw/`
3. `"Ingest the source I just added to raw/"`
4. `"Run wiki-self-heal"` once a week

That's the whole loop.

## Why this exists

Most people's AI workflow: paste context into a chat, get an answer, start a new chat, paste everything again. Knowledge evaporates between sessions. Nothing compounds.

The wiki pattern is different. The LLM reads every source **once**, extracts the knowledge, and writes it into an interlinked set of markdown files — a persistent wiki that sits between you and your raw sources. When you ask a question, it reads the wiki (not the sources). When you add a source, it integrates, flags contradictions, and strengthens the synthesis. Knowledge is compiled once and kept current.

No vector database. No embeddings. No chunking pipeline. The folder is the app.

## How it works

A three-layer mental model:

1. **The map** — `CLAUDE.md` at the vault root. The AI reads this first, every time. It contains the folder structure, the routing table ("for task X, read these files"), naming conventions, workflows, and guardrails. Think of it as the floor plan posted at the entrance of a building.
2. **The rooms** — `raw/` (immutable sources, read-only) and `wiki/` (LLM-owned markdown). The map tells the AI which room to enter for which task. Each room has its own rules.
3. **The workspace** — the actual markdown files inside each room. Pages linked by `[[wikilinks]]`, cataloged in `wiki/index.md`, timelined in `wiki/log.md`.

Three operations: **ingest** (read source → write into wiki), **query** (answer questions from the wiki), **lint** (find and fill gaps). The `llm-wiki-setup` skill installs the whole structure. The `wiki-self-heal` skill runs the lint loop on demand or on a schedule.

```
<your-vault>/
├── CLAUDE.md              # the map — AI reads this first
├── AGENTS.md              # mirror for Codex and other agents
├── raw/                   # immutable sources (LLM reads, never writes)
└── wiki/                  # LLM-owned markdown
    ├── index.md           # catalog of every page
    ├── log.md             # chronological operation log
    └── <pages>.md         # entity / concept / analysis pages
```

## Install

```bash
git clone https://github.com/NulightJens/ai-second-brain-skills.git ~/ai-second-brain-skills
ln -s ~/ai-second-brain-skills/llm-wiki-setup  ~/.claude/skills/llm-wiki-setup
ln -s ~/ai-second-brain-skills/wiki-self-heal  ~/.claude/skills/wiki-self-heal
```

Symlinks mean `git pull` in `~/ai-second-brain-skills` updates both skills instantly. Restart Claude Code after the first install.

**Smoke test** — in Claude Code from any directory:

> "What skills do you have available for building an LLM wiki?"

You should see `llm-wiki-setup` and `wiki-self-heal` in the response.

## Use it

### 1. Set up a vault

From Claude Code, anywhere:

> "Set up an LLM wiki at `~/research-vault`"

The skill asks three questions (path, flat or nested layout, hot cache yes/no), then scaffolds the vault and initializes git.

### 2. Ingest a source

Drop a PDF, web clip, transcript, or markdown file into `~/research-vault/raw/`. Then from inside the vault:

```bash
cd ~/research-vault && claude
```

> "Ingest the new source I just added to raw/"

Claude reads it, creates or updates wiki pages, updates the index, and appends to the log. A single substantive source typically touches 10–15 pages — that is normal and the whole point.

### 3. Query

> "Where do my sources agree and disagree about `<topic>`?"
> "What are the gaps in my wiki right now?"
> "Write a comparison page for X vs Y and file it back into the wiki."

Good answers get filed back as new pages — exploration compounds along with sources.

### 4. Let it learn on repeat

On demand:

> "Run wiki-self-heal on this vault"

**First run: use audit-only mode.**

> "Run a dry-run audit of this vault — don't change anything"

The first pass on a populated wiki usually finds many gaps. Read the audit report first, then decide priorities.

For recurring runs, schedule weekly — see [wiki-self-heal/references/scheduling.md](wiki-self-heal/references/scheduling.md) for launchd, cron, GitHub Actions, and Claude Code trigger recipes.

**Never schedule a skill you haven't run manually first.**

## Requirements

- [Claude Code](https://claude.com/claude-code) CLI, installed and authenticated
- `git` (for wiki history and self-heal branches)
- Optional: [Obsidian](https://obsidian.md) — the vault is plain markdown, any editor works, but Obsidian's graph view is delightful
- Optional for `wiki-self-heal`: Apify, Firecrawl, or Exa MCPs for higher-quality research. Defaults to WebSearch + WebFetch, which always work.

## When to use something else

The wiki pattern is a **personal-to-small-team** tool. At hundreds of pages, it beats vector-DB RAG on token efficiency, simplicity, and transparency — you can literally read your own index, the AI does the same. At millions of documents or many concurrent users, traditional RAG still wins. Use that instead.

Rule of thumb: if you can read your own `wiki/index.md` in under two minutes, you're in the sweet spot for this pattern.

## Credits

- Pattern and original gist by [Andrej Karpathy](https://github.com/karpathy) — [LLM wiki gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Research loop structure inspired by Karpathy's [autoresearch](https://github.com/karpathy/autoresearch)
- Routing-table and folder-as-workspace framing inspired by Nate Herkelmann's folder-architecture video
- Skill packaging by [@NulightJens](https://github.com/NulightJens)

## License

MIT

# Agentic OS — Planning Brief
> Self-contained context for starting a planning session. Read this fully before asking questions.

---

## What We Are Building

A personal **Agentic OS** — a system that turns daily workflows into codified, repeatable, optimizable skills, backed by a structured memory layer and eventually a non-terminal UI. The goal is not to use Claude ad-hoc. The goal is to build a system that runs consistently, scales to others, and gets better over time.

**The three layers:**
1. **Architecture** — skills and automations (the actual value)
2. **Memory** — Obsidian vault with a structured folder system and CLAUDE.md
3. **Observability** — a dashboard that wraps skills as buttons for non-terminal users

**Current scope:** We are starting with the memory layer (the wiki) and the skill architecture. The dashboard comes later once skills exist worth exposing.

---

## Layer 1: Skill Architecture

### Core Concept
Everything you do regularly should be codified into a **skill** — a fixed, repeatable prompt that produces consistent output. Skills are organised by domain.

```
Domains → Tasks → Skills → (optional) Automations
```

### Domains
Break your work into areas (examples: research, content, ops, community, personal). Each domain contains discrete tasks. Each task that recurs becomes a skill.

### Skill Design Principles

**Monolith skills (old pattern — avoid for complex/frequent tasks):**
- Single skill file with pros-chaining (step 1, step 2, step 3...)
- Everything runs in the main context window
- Fine once, problematic when run repeatedly — context compounds across runs
- 51k+ tokens per run at scale

**Efficient skill pattern (new — use for any multi-step or frequently-run skill):**

Three techniques, used together:

#### 1. Context Fork
Declared in the skill's YAML front matter. Runs the entire skill in an isolated sub-agent. Intermediate data and tool responses are discarded on exit. Only the useful output returns to the main conversation.

```yaml
# skill front matter example
name: research-lead
description: Researches a LinkedIn profile and outputs a scored brief
model: opus  # or general-purpose to inherit default
fork: true   # runs in isolated sub-agent
```

#### 2. File Handoff
Each step in the chain writes only the fields the next step needs into a minimal JSON file in a `/temp` directory. The next step reads only that file.

- Full LinkedIn scrape → 8,000 tokens of noise → distilled to `profile.json` → ~200 tokens
- Each step produces one small file. Each step consumes only the file it needs.
- Schedule a cron to clear `/temp` regularly.

**Example chain:**
```
scrape_profile → profile.json
enrich_company → company.json
build_signals  → signals.json
score_lead     → score.json
write_outreach → outreach.json
push_to_sheets ← reads all above
```

#### 3. Shell Command Injection (`!` + backticks)
A placeholder inside the skill file that injects file contents programmatically before Claude runs — zero token cost.

```
!`cat temp/profile.json`
```

Shell runs first, substitutes the file output into the skill at that position. Claude never "reads" the file — it just receives the pre-injected content. Keeps skills dynamic without burning reasoning tokens.

**Result of applying all three:** ~85% reduction in context burn (51k tokens → 5–8k tokens per run).

### Skill vs Agent
- **Skill** = the task / operating procedure (what to do, in what order)
- **Agent** = the behavior (voice, persona, decision rules)
- Only create an agent if the same behavior needs to be reused across multiple different skill systems
- Otherwise embed examples and style references directly inside the skill

### Which Skills Get the Full Treatment?
Not every skill needs fork + file handoff + commands. Target:
- Skills with 3+ chained steps
- Skills run more than once a week
- Skills that call external tools (scrapers, APIs, search)

Simple one-shot skills can stay as monoliths.

---

## Layer 2: Memory (Obsidian Vault)

### Structure
Obsidian is just a UI over markdown files. The file structure is the actual value.

Karpathy-inspired baseline:
```
vault/
├── raw/       ← staging area, dumps, unprocessed notes
├── wiki/      ← distilled articles and reports from raw
└── output/    ← deliverables (decks, briefs, posts, etc.)
```

Extend with domain subfolders as needed:
```
vault/
├── raw/
├── wiki/
│   ├── research/
│   ├── ops/
│   └── ...
└── output/
```

The structure only needs to make sense to you and be legible to Claude. There is no universally correct layout.

### CLAUDE.md — Required
A `CLAUDE.md` file at the vault root that:
- Explains the vault's purpose
- Maps every folder to its intent
- Is automatically prepended to every prompt Claude receives

Without this, Claude navigates the vault by guessing. With it, Claude knows exactly where data flows, where to write output, and where to look for context — reducing token waste and errors.

**Minimum CLAUDE.md sections:**
- What this vault is for
- Folder map with one-line descriptions
- Conventions (naming, date formats, tagging)
- What Claude should NOT modify or delete

---

## Layer 3: Observability Dashboard (Future)

### What It Is
A Streamlit app (`app.py`) that wraps skills and automations as clickable buttons. No terminal required. Fires a headless Claude instance using the `--print` flag.

### Why It Matters
- Non-technical team members or clients can execute skills
- Visibility into usage, vault changes, skill run frequency
- Skills only compound in value when others can use them

### Design System (from reference screenshots)
JetBrains Mono terminal cockpit aesthetic. Key components:
- Header: system name + live status indicator + vault name
- Hero cards: key metrics (usage window, routines run, etc.)
- Skill buttons strip: one button per exposed skill/automation
- Output feed: inline result from headless Claude run
- Right panel: vault recent changes, forecasts

**Defer this layer** until the skill architecture and wiki structure are solid.

---

## Key Prompting Rule
Always append **"as of today"** or **"in 2026"** to any prompt involving research or architecture decisions. Claude's training lags behind current tooling — it will recommend outdated patterns unless forced to check current sources.

---

## Adaptation Notes for This Project

This system was designed around a content/agency workflow (lead research, YouTube research, morning scans). The adaptation target here is a **personal wiki** — the emphasis shifts:

| Original focus | This project's focus |
|---|---|
| Lead research pipeline | Knowledge capture and distillation |
| LinkedIn scraping | Raw note → wiki article conversion |
| Google Sheets output | Structured wiki output |
| Daily automations | On-demand research and synthesis skills |
| Team/client handoff | Personal use first, extensible later |

The architecture principles are identical. The domains and tasks will be different.

---

## Open Decisions for This Planning Session

### Architecture
- [ ] What are the 3–5 primary domains for this wiki? (e.g. research, projects, learning, ops)
- [ ] What are the 2–3 most frequent tasks per domain?
- [ ] Which tasks are run often enough to warrant the fork + file pattern?
- [ ] Which tasks are good candidates for automation vs. on-demand?

### Memory
- [ ] What folder structure fits this wiki's actual use?
- [ ] What should the `CLAUDE.md` say about how Claude should behave inside this vault?
- [ ] What naming and tagging conventions to enforce?

### Skills to Build First
- [ ] What is the highest-value skill to start with? (the one that, if automated, saves the most time)
- [ ] What does a raw note → wiki article skill look like for this project?
- [ ] Are there any morning/daily automations worth building early?

### Dashboard
- [ ] Is a non-terminal UI needed now, or is terminal-first fine while skills are being built?
- [ ] If yes — which skills become the first buttons?

---

## Suggested Starting Point

1. Define your domains and list tasks under each
2. Design the vault folder structure
3. Write the `CLAUDE.md`
4. Build the first skill (simplest high-value task)
5. Test, measure context cost, refine
6. Add fork + file pattern when skills get complex
7. Dashboard last

---

## Source Material
Condensed from two video transcripts on Claude Code Agentic OS design:
- Video 1: Three-step Agentic OS (architecture → memory → observability)
- Video 2: Skill chaining efficiency (fork, file handoff, shell command injection)

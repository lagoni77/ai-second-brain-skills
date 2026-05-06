# Agentic OS — Planning Brief (v2)
> Self-contained context for starting a planning session. Read this fully before asking questions.
> Sources: four video transcripts covering architecture, skill chaining, consultant delivery model, and common myths/anti-patterns.

---

## What We Are Building

A personal **Agentic OS** — a system that turns daily workflows into codified, repeatable, optimizable skills backed by a structured memory layer and eventually a non-terminal UI. The goal is not to use Claude ad-hoc. The goal is a system that runs consistently, can be handed to others, and gets better over time.

**The three pillars (non-negotiable):**
- **Reliable** — runs the same way every time
- **Accurate** — produces correct output against a defined definition of good
- **Predictable** — the path from input to output is known before the skill runs

Everything else is secondary to these three. Complexity that doesn't serve them should be cut.

---

## Core Architecture: Domains → Tasks → Skills

Break your work into **domains** (e.g. research, content, ops, community). Each domain has discrete **tasks**. Each recurring task becomes a **skill**.

```
Domains → Tasks → Skills → (optional) Automations
```

### Skills vs Agents — A Hard Line

This distinction matters more than most videos admit:

| Skills | Agents |
|---|---|
| Known, predictable path | Unknown path, goal-driven |
| Step 1 → Step 2 → done | Here are tools, figure it out |
| Reliable, auditable | Flexible, harder to maintain |
| Right choice for 90%+ of business workflows | Right choice when you genuinely can't define the steps |

**Default to skills.** If you can write down what you do step-by-step, it's a skill, not an agent use case. Agents are for exploration and edge cases — not production workflows. Elaborate agent front-ends with no predictable path are the most common mistake in the space right now.

---

## Skill Design: Efficiency Patterns

### Monolith (avoid for complex/frequent tasks)
- Everything in one skill file, pros-chaining step 1→2→3
- Runs in main context window
- Fine once, compounds badly across repeated runs (~51k tokens per run at scale)

### Efficient Pattern (use for 3+ steps or frequent runs)

Three techniques used together:

#### 1. Context Fork
YAML front matter setting. Runs the skill in an isolated sub-agent. Intermediate tool responses are discarded on exit. Only the useful output returns to the main conversation.

```yaml
name: skill-name
description: What this skill does
fork: true
model: general-purpose  # inherits default, or specify
```

#### 2. File Handoff
Each step writes only what the next step needs into a minimal JSON file in `/temp`. Cuts raw data noise to ~200 tokens per handoff.

```
step_1 → temp/profile.json      (~200 tokens)
step_2 → temp/signals.json      (~150 tokens)
step_3 → temp/score.json        (~50 tokens)
final  ← reads all, pushes output
```
Schedule a cron to clear `/temp` regularly.

#### 3. Shell Command Injection
Injects file contents into a skill at parse time — zero token cost, no Claude reasoning required.

```
!`cat temp/profile.json`
```

**Result:** ~85% context reduction (51k → 5–8k tokens per run).

### Skill Portability Principle
Context, references, and examples that a skill needs should live **inside the skill folder**, not in an external system. This makes skills portable — lift, shift, and tailor for a new environment without external dependencies.

---

## Memory: Three Separate Things

This is the most misunderstood layer. Memory is not one thing. It has three distinct components that belong in different places.

### 1. Knowledge
What Claude needs to know to act on your behalf: your voice, business context, who you serve, what good output looks like. Lives in:
- `CLAUDE.md` at the project root (global context, prepended to every prompt)
- Reference files and examples inside each skill folder (local context, loaded per skill)
- External docs system (Notion, Confluence, etc.) accessed via MCP when needed globally

### 2. State
The current status of things moving through workflows — lead pipeline stage, which notes have been processed, task completion tracking. Lives in:
- A simple database: Airtable, SQLite, a spreadsheet
- **Not** markdown files — databases were built for this, markdown was not

### 3. Learned Memory
What Claude discovers about you through working together — patterns, preferences, corrections. Lives in:
- Claude's own memory feature — let it build organically
- Supplemented by explicit rules added to `CLAUDE.md` when a pattern repeats

### What About RAG?
Most people will never need it. RAG is for semantic search across thousands of documents. For a personal wiki or small team:
- Claude reads markdown files directly — no vector database needed
- Keyword matching in a database handles most "find this thing" use cases
- Hybrid RAG (mixing semantic + keyword) is what actually works reliably in production, which makes it complex to set up correctly

**Rule:** Only reach for RAG if you've hit a concrete wall where Claude can't find what it needs from direct file reads or database queries.

---

## What Obsidian Is (and Is Not)

This is worth stating clearly because it is widely oversold.

**What Obsidian actually does:**
- Provides a UI over local markdown files
- Backlinks, graph view, and semantic search are for **the human**, not for Claude
- Claude does not use backlinks. Claude does not use the graph. There is no reliable MCP for semantic search via CLI.

**When Obsidian makes sense:**
- You personally want a visual layer to browse and read your knowledge base
- You prefer it over Notion as a human interface
- Your use case is personal (not team — Obsidian is poor for collaboration)

**When it does not make sense:**
- You are building it for a business or team (Notion wins on collaboration)
- You expect Claude to use its graph/search features — it won't
- You are migrating out of a working system just because a YouTube video said to

**For the wiki project specifically:** Obsidian is fine as the human-readable interface for browsing the wiki. But the skills that generate wiki content should carry their own context internally, and state tracking (e.g. which raw notes have been converted) should live in a simple database, not in Obsidian markdown.

---

## Constraint-First Build Order

Start from actual pain, not from a template. The architecture should fall out of your answers to these questions — not the other way around.

**Foundation questions:**
1. What workflows do you actually run? (maps to domains and tasks)
2. How many people need to use this? (1 person = simplest path)
3. Where does your knowledge currently live? (don't move it unless necessary)
4. Can you manage this yourself or do you need someone to maintain it?
5. Do any tasks need to run while your computer is off? (determines infrastructure)

**Build order:**
1. Define domains and list the tasks you actually do
2. Write the `CLAUDE.md` — knowledge layer first
3. Build one skill around your highest-friction recurring task
4. Test it. Measure context cost before and after.
5. Add fork + file pattern when the skill has 3+ steps or runs frequently
6. Add state tracking (database) when you need to track things across runs
7. Add automations only for tasks that clearly should run on a schedule
8. Dashboard / non-terminal UI last — only when skills are stable and others need access

---

## Infrastructure: Where Does This Run?

Relevant once you have skills worth running on a schedule.

| | VPS (Hetzner, DigitalOcean) | Mac Mini | Cloud (Routines) |
|---|---|---|---|
| Always on | Yes | Only if powered | Yes |
| Backups | Automatic from host | Manual / Git | N/A |
| Scale | Upgrade plan | Buy new hardware | Scales automatically |
| Cost | ~$5–20/mo | One-time hardware | Included in Claude plan |
| Best for | Most people starting out | Tech-savvy, local models, on-prem data | Simple scheduled prompts (still in research preview) |

For personal use / wiki project: your local machine is fine to start. A VPS becomes relevant when you want skills running on a schedule while your laptop is closed.

**On Routines (cloud-native scheduling):** Currently in research preview. Write permissions are granted to all connected tools with no per-tool granularity — use cautiously. Worth revisiting as it matures.

---

## Distribution (When Others Need Access)

If skills need to reach non-technical users:

1. Store all skills in a **GitHub repo** (also serves as backup)
2. Add that repo as a plugin source in Claude.ai → Customize → Plugins
3. Users access skills via Claude.ai (Cowork) — no terminal, no IDE
4. You manage the backend (VS Code / CLI on VPS or Mac Mini), they use the front end

This creates a clean separation: you own the infrastructure and skill quality, they consume the output.

---

## Vendor Lock-in: The Real Picture

Skills are just documented operating procedures. They run in Claude Code, Codex, Gemini — any provider with a CLI and tool use. The context lives in markdown or in the skill folder. The state lives in a database you own. The GitHub repo is yours. There is no meaningful lock-in for a personal or small-team system.

---

## Differences from Original Plan (v1)

The original plan assumed Obsidian as the memory layer and had a template-first build order. These videos surface important corrections:

| v1 assumption | Corrected view |
|---|---|
| Obsidian is the memory layer | Memory has three components; Obsidian is a UI, not a memory system |
| raw/wiki/output folder structure | Structure should emerge from your actual workflows, not a template |
| Agent vs skill is a nuance | It is a hard architectural decision — predictable path = skill, always |
| Dashboard is Layer 3 | Dashboard is Layer 3 AND requires stable skills to be worth building |
| Context lives in vault | Context lives inside each skill's own folder for portability |
| RAG mentioned as advanced option | RAG is rarely needed — most personal wikis don't qualify for it |

**What stays the same:**
- Domains → Tasks → Skills → Automations flow
- Context fork + file handoff + shell injection for efficiency
- CLAUDE.md is required
- Skill vs Agent distinction
- Dashboard uses headless Claude via `--print`

---

## Open Decisions for This Planning Session

### Foundation
- [ ] What are your actual domains? (start with what you do today, not what you aspire to)
- [ ] What is the single highest-friction recurring task right now?
- [ ] Where does your knowledge currently live — and should it move?

### Memory Architecture
- [ ] What goes in `CLAUDE.md` vs inside individual skill folders?
- [ ] What things need state tracking? What is the simplest database that works?
- [ ] Is Obsidian the right human interface for browsing this wiki, or is something else better?

### Skill Design
- [ ] What does the raw → wiki article conversion skill look like step by step?
- [ ] How many steps does it have? Does it qualify for the fork + file pattern?
- [ ] What is the definition of a good wiki article for this project?

### Infrastructure
- [ ] Does anything need to run while your laptop is closed?
- [ ] Is a VPS needed now, or is local machine fine to start?

### Growth
- [ ] Will anyone else need to use this system?
- [ ] If yes — does the plugin marketplace route make sense, or is terminal access fine?

---

## Key Prompting Rule
Always append **"as of today"** or **"in 2026"** to research or architecture prompts. Claude's training lags — it will recommend outdated patterns (e.g. treating MCP as always-bloating context, which lazy loading has since addressed) unless forced to check current sources.

---

## Source Material
Condensed from four video transcripts:
- Video 1: Three-step Agentic OS (architecture → memory → observability)
- Video 2: Skill chaining efficiency (fork, file handoff, shell command injection)
- Video 3: Consultant engagement model (ATOM framework, VPS vs Mac Mini, plugin distribution)
- Video 4: Three myths (agents vs skills, memory components, Obsidian limitations)

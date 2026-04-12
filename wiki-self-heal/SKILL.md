---
name: wiki-self-heal
description: Autonomously audit an LLM wiki (Karpathy pattern) for gaps, contradictions, orphans, and stale data, then research and fill high-priority gaps using quality-gated web research. Runs on schedule or on demand. Operates on a dedicated branch and commits changes for human review — never auto-merges. Use when the user asks to "lint my wiki", "self-heal my knowledge base", "find gaps in my wiki", "update my second brain", "auto-research my wiki", "run a health check on my LLM wiki", or wants to schedule periodic wiki maintenance.
---

# wiki-self-heal

Audit → research → fill → commit. An autonomous pass over a Karpathy-style LLM wiki that finds what's weak or missing, researches high-quality answers, writes them into the wiki, and commits to a dedicated branch for human review.

Inspired by Karpathy's `autoresearch` loop structure: isolated branch per run, clear CAN/CANNOT rules, numbered steps, explicit logging, simplicity criterion, and a NEVER STOP autonomy directive once the loop begins.

## When to use

User says: "lint my wiki", "self-heal", "find gaps", "auto-research my wiki", "update my second brain", "health check my wiki", or has scheduled this skill via cron / GitHub Actions / Claude Code triggers.

## Preconditions

Before doing anything, verify:

1. The vault has Karpathy wiki structure: `raw/`, `wiki/`, `CLAUDE.md`, `wiki/index.md`, `wiki/log.md`. If any are missing → stop and suggest running the `llm-wiki-setup` skill first.
2. The vault is a git repo. If not → `git init && git add . && git commit -m "chore: pre-heal snapshot"` before continuing.
3. The git working tree is clean. If dirty → stop and ask the user to commit or stash first.

## Research tool selection

Ask **once** at the start of a session:

> Do you have any of these MCPs installed for higher-quality research? (a) Apify, (b) Firecrawl, (c) Exa. Reply with letters or "none".

Default to **WebSearch + WebFetch** (always available). If the user answers with MCPs, prefer them in this priority: Exa → Firecrawl → Apify → WebSearch/WebFetch. Fall back to the next tool on any failure.

## The loop

Full procedure in [references/loop.md](references/loop.md). Core sequence:

1. **Read state** — `CLAUDE.md`, `wiki/index.md`, last 10 entries of `wiki/log.md`.
2. **Create branch** — `git checkout -b wiki-heal/$(date +%Y-%m-%d)`. Append `-2`, `-3`, etc. if a branch for today already exists.
3. **Audit** — scan for the 6 gap types. Procedures and severity rubric in [references/audit.md](references/audit.md). Write findings to `wiki/audits/audit-<date>.md`.
4. **Prioritize** — pick top **N** (default N=3) high-severity, skill-addressable gaps. Skip gaps needing human judgment (e.g. contradictions between authoritative primary sources — flag for human).
5. **Research each gap** — apply the quality gates in [references/research-quality.md](references/research-quality.md). Every factual claim needs ≥ 2 independent sources. Reject SEO spam and content farms.
6. **Apply** — create or update wiki pages with cited findings. Update `wiki/index.md`. Add bidirectional `[[wikilinks]]`.
7. **Log** — append one consolidated entry to `wiki/log.md` per [references/log-format.md](references/log-format.md).
8. **Commit** — `git add wiki/ && git commit -m "heal: <n> gaps addressed, <s> skipped"`. **Leave on branch. Do not merge.**
9. **Report** — print the final summary table.

## CAN / CANNOT

**CAN**:
- Create new wiki pages
- Update existing wiki pages
- Update `wiki/index.md`
- Append to `wiki/log.md`
- Create `wiki/audits/audit-<date>.md`
- Download new sources to `raw/` if research pulls in a fresh document (note in log)
- Create a heal branch and commit to it

**CANNOT**:
- Modify any existing file in `raw/`
- Delete any wiki page (always flag for human review instead)
- Add any factual claim without ≥ 2 qualifying sources
- Merge the heal branch into main
- Skip the log or commit step
- Pause the loop to ask "should I continue?" (see Autonomy rule below)

## Simplicity criterion

All else equal, **fewer new pages wins**. A 1-source stub that only restates what's already in `[[related-page]]` should be skipped or merged into the existing page, not created as its own file. Prefer updating existing pages over creating new ones when a concept already has a home. Adding 20 thin orphan pages is a regression, not progress.

## Autonomy rule

Once the loop begins (after preconditions and tool selection), **do not pause to ask the human if you should continue**. The human may be asleep, away, or expecting autonomous operation. Work through the top-N gaps, then stop and report. Only stop early if:

- A CANNOT rule would be violated
- Quality gates fail for all sources on a gap → skip that gap, log reason, continue
- The heal branch has drifted from main by more than 50 commits → stop and report
- A git operation fails → stop and report

## Scheduling

For recurring runs (recommended: weekly for active vaults, monthly for slow vaults), see [references/scheduling.md](references/scheduling.md). Three recipes: macOS cron, GitHub Actions for cloud vaults, and Claude Code scheduled triggers.

**First run guidance**: always run `wiki-self-heal` manually at least once and review the resulting heal branch before automating it. Never schedule a skill you haven't run manually first.

## Output format

On completion, print exactly this report:

```
Wiki self-heal pass — <YYYY-MM-DD>
Branch: wiki-heal/<tag>

Audit: <N> gaps found (HIGH:<h> MEDIUM:<m> LOW:<l>)
Addressed: <n>
Skipped: <s>

New pages:
  - [[<page>]] — <one-line summary>
Updated pages:
  - [[<page>]] — <what changed>

Sources added: <k>
Commit: <short-hash>

Review: git diff main..wiki-heal/<tag>
Merge:  git checkout main && git merge wiki-heal/<tag>
```

---
name: disney-model
description: >
  Run a structured Disney Creative Strategy session on any project, problem, or
  decision. Launches three sequential thinking modes — Dreamer (blue-sky vision),
  Realist (practical execution), Critic (risk analysis) — and synthesises their
  outputs into a Kanban board, prioritised action list, and optional iteration loop.
  Use when you need to evaluate a project holistically, when a plan feels stuck,
  when you want to stress-test an idea before committing resources, or when you
  need a structured second opinion on direction.
  Trigger: "kør Disney Model på [projekt/problem]", "Disney analyse", "Dreamer
  Realist Critic på [emne]", or "stress-test dette".
---

# disney-model

Structured brainstorming technique developed by Robert Dilts (1994). Separates
three thinking modes that normally interfere with each other into distinct,
sequential roles. Each role gets full attention — no premature filtering of ideas,
no unchallenged optimism, no criticism without alternatives.

---

## The Three Roles

| Role | Core question | Mindset |
|------|--------------|---------|
| **The Dreamer** | What could this become? | No limits. No filters. What if anything were possible? |
| **The Realist** | How do we make this happen? | Practical. Resource-aware. Sequential. Honest about time. |
| **The Critic** | What could go wrong? | Constructively sceptical. Specific risks. Concrete mitigations. |

---

## Inputs

The user provides one of:
- A **project name** and brief context (Claude reads the relevant files)
- A **problem statement** ("we need to decide whether to X")
- A **plan or document** to evaluate

Optionally:
- `[FOCUS]` — which role's output matters most for this session
- `[DEPTH]` — `quick` (3-5 points per role) or `deep` (8-12 points per role, default)
- `[ITERATION]` — `yes` to loop Critic's output back to Dreamer for refinement

---

## Process

### Phase 1 — Context read
Before generating any role output, read the relevant project files. Understand what exists, what's been decided, and what the user is trying to evaluate.

### Phase 2 — The Dreamer 🧙
Generate bold, specific, unfiltered ideas. Rules:
- No practicality filter — if it requires technology that doesn't exist yet, say so and include it anyway
- Concrete enough to visualise, not vague enough to be useless ("better user experience" is banned)
- 8-12 ideas for `deep`, 3-5 for `quick`
- Format: **Bold headline** — 2-3 sentence description

### Phase 3 — The Realist 📋
Assess the practical path. Deliver:
1. **Current state** — what's done, partially done, not started (be honest)
2. **Realistic workflow** — step-by-step process with time estimates
3. **Bottlenecks** — the 3-5 things that will actually slow execution
4. **Sequencing** — what must happen before what (dependencies)
5. **Effort estimate** — per unit and total

### Phase 4 — The Critic ⚠️
Identify risks and gaps. For each issue:
- **Issue** — what is wrong or missing (be specific, not vague)
- **Risk level** — 🔴 High / 🟡 Medium / 🟢 Low
- **Mitigation** — one concrete, actionable fix

8-12 issues for `deep`, 3-5 for `quick`. Address: legal risks, data quality, technical fragility, workflow gaps, missing documentation, single points of failure.

### Phase 5 — Synthesis
Produce:
1. **Kanban board** — Done / In Progress / Backlog / Blocked
2. **Top 3 immediate actions** — highest-leverage moves right now, in priority order
3. **Top 3 Dreamer ideas worth pursuing** — filtered through Realist/Critic lens

### Phase 6 — Iteration (if requested)
Feed the Critic's top concerns back into a focused Dreamer pass: "Given these risks, what creative solutions could address them?" Then run a final Realist check on the proposed solutions.

---

## Output format

```
---
## 🧙 THE DREAMER
[ideas]

---
## 📋 THE REALIST
[assessment]

---
## ⚠️ THE CRITIC
[risks]

---
## SYNTHESIS
### Kanban
[board]

### Immediate Actions
1. ...
2. ...
3. ...

### Dreamer Ideas Worth Pursuing
1. ...
2. ...
3. ...
```

---

## Guardrails

- Each role must stay in character — the Dreamer does not add caveats, the Critic does not propose alternatives without also identifying what's broken
- Never merge roles in a single paragraph — separation is the point
- Criticism must be constructive: every risk must have a mitigation
- The Realist must include time/effort estimates — vague "this will take time" is not useful
- If the user only wants one role, run only that role but note which perspective is missing

---

## When to iterate

Run the full loop (Dreamer → Realist → Critic → back to Dreamer) when:
- The plan survives the first Critic pass but still feels incomplete
- The Critic identified a fundamental flaw that changes the vision
- The user explicitly asks for iteration

One pass is usually enough for status reviews and project health checks.
Two passes are appropriate for major decisions or course corrections.

---
name: adaptive-team-consult
description: "Consult the team on a topic — PM selects the right specialists, they brief and discuss. No code is written."
argument-hint: "<topic or question>"
---

# Adaptive Team Consult

PM brings in the specialists whose lenses fit the topic and facilitates a plain-English discussion. No code is written; the output is shared understanding and a direction.

## Pipeline

### Step 0: PM Recovery / Grounding

PM reads, in order:
1. `.claude/rules/adaptive-team-rules.md`
2. `.claude/adaptive-team-state/current-session.md` (create from template if missing)
3. `.claude/adaptive-team-learned/pm-lessons.md`
4. `TaskList`

### Step 1: Classify the Topic

PM identifies topic signals and selects reviewers per the Consult Mode rubric in the rules:

| Signal | Spawn |
|--------|-------|
| Schema, query, migration, index, graph | `database` |
| Prompt, tool use, model choice, agent design, eval | `llm-expert` |
| Auth, permissions, authz, secrets | `architect` |
| Test strategy, coverage, flake | `sdet` |
| Architecture / coherence | `architect` |
| Ambiguous | `architect` as default |

Default minimum: `architect`. Spawn only those the signal warrants — do not invite everyone by habit.

### Step 2: Create or Join Team

If a team exists, join it — do not recreate. Otherwise `TeamCreate` with a descriptive name. Update `current-session.md`.

### Step 3: Spawn Selected Reviewers

Use the deterministic names: `architect`, `sdet`, `llm-expert`, `database`. Each completes its startup sequence.

### Step 4: Facilitate the Consult

PM forwards the user's topic to spawned reviewers and asks each for:
- Their plain-English read
- Risks or constraints specific to their lens
- Recommended direction (not code)

Each reviewer writes a full analysis to `.claude/adaptive-team-reviews/consult/<short-slug>/<reviewer>.md` and returns a short summary to PM. PM synthesizes for the user by showing **one paragraph per reviewer, verbatim from the summary line** — PM does not re-author.

### Step 5: Decide or Iterate

User decides the direction, asks follow-ups, or requests `/adaptive-team-implement` to build.

### Step 6: Update Session State

PM updates `current-session.md` with any key decisions and the set of spawned reviewers (which remain persistent for this session).

## When to Use

- Approach or design questions before implementation
- Choosing between options (model, schema, framework)
- Understanding risk and tradeoffs on a specific topic

## When NOT to Use

- Actual implementation → `/adaptive-team-implement`
- Capturing a lesson from something that went wrong → `/adaptive-team-learning-moment`

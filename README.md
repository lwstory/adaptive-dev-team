# Adaptive Dev Team

A self-healing Claude Code agent team for structured software development. The team adapts to your project (via init) and adapts to mistakes (via human-in-the-loop learning). Every lesson is user-approved before it lands.

## What This Is

Agent definitions, rules, skills, and templates that spin up a complete development team:

| Role | Purpose |
|------|---------|
| **adaptive-team-product-manager** (PM) | Orchestrates on the main thread — epics, stories, delegation, acceptance. Never reads or writes code. Receives verdicts only. |
| **adaptive-team-architect** | Design coherence, permissions/authz review, dev-model selection per task. Always spawned. |
| **adaptive-team-sdet** | Test strategy via behavior-ID briefings, coverage-hole gate, pyramid discipline. Always spawned when code is written. |
| **adaptive-team-llm-expert** | Prompt engineering, model selection, structured-output contracts, prompt-injection trust boundaries. On-demand. |
| **adaptive-team-database** | Schema/query design across MySQL, Neo4j, and document stores; query-plan artifact gate. On-demand. |
| **adaptive-team-curious** | Generalist ideator — "I wonder if...", "What if we...", "Could we try..." Produces improvement proposals vetted by specialists. On-demand. |
| **adaptive-team-dev** | Implementation, testing, commits. Task-scoped, fresh per task, always in a worktree. |

## Key Features

- **PM context discipline** — PM never reads/edits source, never ingests review findings. Enforced by tool palette + strict 3-line verdict format. Keeps main thread lean; survives compaction.
- **Deterministic names** — `architect`, `sdet`, `llm-expert`, `database`, `curious`, `dev` — addressable across compaction.
- **Shift-left briefings** — relevant reviewers brief the dev in plain English before code is written. Architect briefings end with a grep-detectable `**Careless-dev failure mode:**` clause and a `## Gotchas` section. SDET briefings are behavior-ID lists (`BH01`, `BH02`, ...) with exactly one `[COVERAGE-HOLE]` tag.
- **Mandatory pre-flight gates** — dev must send a 3-bullet pre-flight echo (B1) and an unknown-unknowns question (B8) and receive confirmation before writing any code.
- **Reviewer self-reflection** — when dev work is UNSATISFIED, the *reviewer* first asks whether its own briefing was adequate. Dev never self-assesses. Lessons route to the reviewer's file or the dev's file based on diagnosis.
- **User-approved learning** — no file under `adaptive-team-learned/` or `~/.claude/pm-user-lessons.md` is written without explicit user approval. Every lesson proposal is forwarded verbatim.
- **User-global vs project-PM lessons** — PM observations about *how to work with the user* (style, tone, timing) live at `~/.claude/pm-user-lessons.md` (cross-project). Project-PM process lessons live in the project. See [Learning routing](#learning-routing).
- **Identity-block contract** — first ~30 lines of every role file are self-sufficient (role, tool constraints, verdict format), so an agent can function after aggressive context compaction.
- **Exhaustive startup** — each role file lists every file it reads on startup with approximate size; context budget is derivable by inspection.
- **Worktree isolation** — every dev works in a git worktree, merges only after review.
- **Review gates** — every relevant reviewer must return SATISFIED before PM accepts (max 2 review cycles → escalate to user).

## Prerequisites

This framework uses [Claude Code Agent Teams](https://docs.anthropic.com/en/docs/claude-code/agent-teams), which enables multi-agent coordination with peer-to-peer messaging and shared task lists.

| Requirement | Details |
|-------------|---------|
| **Claude Code** | v2.1.32 or later (`claude --version`) |
| **Agent Teams** | Experimental — must be enabled (see below) |
| **Git** | Required — worktrees are used for dev isolation |

### Enable Agent Teams

`/adaptive-team-init` checks for this and offers to enable it. To set it up manually:

```json
// ~/.claude/settings.json or .claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Setup

### 1. Install the init skill (once, globally)

**macOS / Linux:**

```bash
mkdir -p ~/.claude/skills/adaptive-team-init
curl -o ~/.claude/skills/adaptive-team-init/SKILL.md \
  https://raw.githubusercontent.com/lwstory/adaptive-dev-team/master/.claude/skills/adaptive-team-init/SKILL.md
```

**Windows (PowerShell):**

```powershell
New-Item -ItemType Directory -Force -Path "$HOME\.claude\skills\adaptive-team-init"
Invoke-WebRequest -Uri "https://raw.githubusercontent.com/lwstory/adaptive-dev-team/master/.claude/skills/adaptive-team-init/SKILL.md" `
  -OutFile "$HOME\.claude\skills\adaptive-team-init\SKILL.md"
```

This makes `/adaptive-team-init` available in any project.

### 2. Initialize in your project

Open your project in Claude Code and run:

```
/adaptive-team-init
```

This does everything in one flow:
1. Checks dependencies (Git, Claude Code version, Agent Teams enabled, git repo)
2. Clones the framework repo to temp
3. Copies agents, rules, and skills into your `.claude/`
4. Seeds runtime directories (`adaptive-team-learned/`, `adaptive-team-state/`, `adaptive-team-reviews/`) from templates — never overwrites accumulated content
5. Seeds `~/.claude/pm-user-lessons.md` (user-global) if missing
6. Auto-detects your tech stack and asks targeted questions
7. Writes project-specific context to `.claude/adaptive-team-context/`
8. Cleans up

### 3. Start building

```
/adaptive-team-consult <topic>          — discuss approach, no code written
/adaptive-team-implement <task>         — full implementation pipeline
/adaptive-team-learning-moment [what]   — capture a lesson from an issue
```

## Skills Reference

| Skill | Scope | What it does |
|-------|-------|-------------|
| `/adaptive-team-init` | Global (install to `~/.claude/skills/`) | Full initialization — dependency checks, file copy, runtime-dir seeding from templates, auto-detect, setup wizard. |
| `/adaptive-team-consult` | Project | PM picks the right specialists for a topic (per the reviewer-selection rubric) and facilitates a plain-English discussion. No code is written. |
| `/adaptive-team-implement` | Project | Full implementation pipeline — PM creates stories, reviewers brief the dev (shift-left), dev builds in a worktree, reviewers sign off, then merge. |
| `/adaptive-team-learning-moment` | Project | Capture lessons. Any reviewer or PM may propose; PM routes through the user for approval; approved text lands in the right file. |
| `/adaptive-team-meta-eval` | Project | **Design stub** — framework-on-framework regression detection via scored scenarios. Not yet runnable; stub describes intended behavior. |

**P2 guard on ambiguous asks:** if acceptance criteria can't be expressed in one sentence, PM auto-routes `/adaptive-team-implement` requests to `/adaptive-team-consult` first.

## How It Works

```
User request
  → PM (P2 check): one-sentence acceptance criteria? if not → /adaptive-team-consult first
  → PM creates stories (TaskCreate)
    → PM spawns relevant reviewers (architect always; sdet when code; llm-expert / database / curious on signal)
      → Architect decides dev model for this task; reviewers brief the dev (shift-left)
        → Dev pre-flight echo (B1) + unknown-unknowns ask (B8) — mandatory gates
          → Dev implements in a worktree
            → Reviewers review: findings direct to dev, verdict (3-line) to PM
              → All SATISFIED → PM accepts → dev commits and merges
                → Retrospective: any issue → /adaptive-team-learning-moment
                  → Reviewer self-reflects: own briefing gap vs dev-side → proposes lesson
                    → PM forwards verbatim → user approves → written to the right file
```

## Architecture

```
.claude/
  agents/                                    ← role identity, Identity Block + Startup sizes
    adaptive-team-product-manager.md
    adaptive-team-architect.md
    adaptive-team-sdet.md
    adaptive-team-dev.md
    adaptive-team-llm-expert.md
    adaptive-team-database.md
    adaptive-team-curious.md
  rules/
    adaptive-team-rules.md                   ← team process, reviewer protocol, contracts
  skills/
    adaptive-team-init/                      ← installer (copies files, seeds runtime)
    adaptive-team-consult/                   ← specialist consult (no code)
    adaptive-team-implement/                 ← full implementation pipeline
    adaptive-team-learning-moment/           ← lesson capture (user-approved)
    adaptive-team-meta-eval/                 ← framework-on-framework eval (design stub)
  templates/                                 ← canonical seeds for init
    current-session.md.template
    pending-pm-lessons.md.template
    reviews-README.md.template
    pm-user-lessons.md.template              ← seeded to ~/.claude/ on install
    <role>-lessons.md.template               ← one per role
  adaptive-team-context/                     ← project knowledge (filled by init wizard)
    tech-stack.md
    architecture.md
    testing.md
    review-checklist.md
    vocabulary.md                            ← 20 load-bearing project terms
  adaptive-team-learned/                     ← runtime, per-project lessons (.gitignored)
  adaptive-team-state/                       ← runtime, session state (.gitignored)
  adaptive-team-reviews/                     ← runtime, per-task findings (.gitignored)
```

**Separation of concerns:**
- **Agent files** = who each role is (stable, rarely changes)
- **Rules** = team process and contracts (versioned per framework release)
- **Context files** = what you're working on (project-specific, populated by init)
- **Templates** = clean starting content for runtime dirs (ship with the framework)
- **Runtime dirs** (`adaptive-team-learned/`, `adaptive-team-state/`, `adaptive-team-reviews/`) = accumulated per-project state. **Not committed to the framework repo.** Your project may or may not commit them — that's your call.

## Learning Routing

Every lesson is user-approved before being written. Routing table (full version in `rules/adaptive-team-rules.md`):

| Root cause | File |
|------------|------|
| User communication, style, question timing, tone | **`~/.claude/pm-user-lessons.md`** (cross-project) |
| Project-PM process, coordination, scope | `.claude/adaptive-team-learned/pm-lessons.md` |
| Design, permissions/authz, coherence | `architect-lessons.md` |
| Test strategy, coverage | `sdet-lessons.md` |
| Prompt, model selection, agent design, LLM failure modes | `llm-lessons.md` |
| Schema, query, migration | `database-lessons.md` |
| Implementation patterns | `dev-lessons.md` |
| Cross-cutting process | `team-lessons.md` |
| Ideation patterns | `curious-lessons.md` |
| Project-wide policy (only when the moment warrants) | `CLAUDE.md` |

**CLAUDE.md never receives promotions from learning files** — additions only when a specific issue at that moment warrants project-wide policy, decided by the user at that moment.

**Staging:** PM writes in-flight observations to `.claude/adaptive-team-state/pending-pm-lessons.md`. At natural boundaries, PM reviews pending entries with the user. Approved entries promote to their target file; rejected entries are deleted.

See also: [`docs/hitl-self-healing.md`](docs/hitl-self-healing.md).

## License

MIT

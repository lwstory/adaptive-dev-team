# Adaptive Dev Team

A self-healing Claude Code agent team for structured software development. The team adapts to your project (via setup) and adapts to mistakes (via human-in-the-loop learning).

## What This Is

Agent definitions, rules, and skills that spin up a complete development team:

| Role | Purpose |
|------|---------|
| **adaptive-team-product-owner** | Orchestrates work — epics, stories, delegation, acceptance |
| **adaptive-team-architect** | Design guidance, code review, architectural quality gate |
| **adaptive-team-sdet** | Test strategy, test review, coverage verification |
| **adaptive-team-dev** | Implementation, testing, commits |

## Key Features

- **Self-healing** — operational failures and user misalignment become permanent lessons
- **Root-cause routing** — lessons go to the right file (PO, architect, sdet, dev, or team)
- **Worktree isolation** — every dev works in a git worktree, merges only after review
- **Review gates** — architect + sdet must both sign off before code is accepted
- **Thin agents** — role files are ~100 lines; project knowledge lives in `adaptive-team-context/`
- **Task-scoped devs** — fresh context per task prevents bias and enables parallelism

## Prerequisites

This framework uses [Claude Code Agent Teams](https://docs.anthropic.com/en/docs/claude-code/agent-teams), which enables multi-agent coordination with peer-to-peer messaging and shared task lists.

| Requirement | Details |
|-------------|---------|
| **Claude Code** | v2.1.32 or later (`claude --version`) |
| **Agent Teams** | Experimental — must be enabled (see below) |
| **Git** | Required — [worktrees](https://git-scm.com/docs/git-worktree) used for dev isolation (isolated working copies on separate branches) |

### Enable Agent Teams

```json
// ~/.claude/settings.json or .claude/settings.json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## Setup

### 1. Copy into your project

```bash
cp -r adaptive-dev-team/.claude/ your-project/.claude/
cp adaptive-dev-team/CLAUDE.md your-project/CLAUDE.md
```

### 2. Run the setup wizard

```
/adaptive-team-setup
```

This scans your codebase, asks targeted questions, and populates `.claude/adaptive-team-context/` with your project's tech stack, architecture patterns, test conventions, and review checklist. Agent files are never modified — all project knowledge lives in context files.

### 3. Start building

Plan first (optional):
```
/adaptive-team-plan Add user authentication with JWT
```

Then implement:
```
/adaptive-team-implement
```

Or skip planning for straightforward tasks:
```
/adaptive-team-implement Fix the broken login redirect
```

## How It Works

```
User request
  → PO creates stories
    → Architect provides design guidance
      → Dev implements in worktree
        → Architect reviews code
        → SDET reviews tests
          → Both SATISFIED → Dev commits and merges
            → PO checks: did the user get what they wanted?
              → Lessons routed to the right file
```

## Architecture

```
.claude/
  agents/                          ← role identity (~100 lines each)
    adaptive-team-product-owner.md
    adaptive-team-architect.md
    adaptive-team-sdet.md
    adaptive-team-dev.md
  adaptive-team-context/           ← project knowledge (from /adaptive-team-setup)
    tech-stack.md
    architecture.md
    testing.md
    review-checklist.md
    project-docs.md
  adaptive-team-learned/           ← accumulated lessons (from HITL self-healing)
    po-lessons.md
    architect-lessons.md
    sdet-lessons.md
    dev-lessons.md
    team-lessons.md
  rules/
    adaptive-team-rules.md
  skills/
    adaptive-team-setup/           ← project setup wizard
    adaptive-team-plan/            ← design review (PO + Architect)
    adaptive-team-implement/       ← full team implementation
docs/
  hitl-self-healing.md
```

**Separation of concerns:**
- **Agent files** = who you are and how you work (stable, rarely changes)
- **Context files** = what you're working on (project-specific, replaceable)
- **Learned files** = what you've learned from mistakes (accumulates over time)

## Self-Healing

When something goes wrong, the PO diagnoses root cause and routes the lesson:

| Root Cause | Lesson File |
|-----------|-------------|
| Requirements/scope miss | `po-lessons.md` |
| Design/architecture gap | `architect-lessons.md` |
| Test coverage/quality gap | `sdet-lessons.md` |
| Implementation pattern issue | `dev-lessons.md` |
| Process/coordination failure | `team-lessons.md` |
| Project-wide policy | `CLAUDE.md` |

Any pattern that repeats twice gets codified. The team gets better with every session. Learned files should be committed to git — they're part of your project's accumulated knowledge.

## License

MIT

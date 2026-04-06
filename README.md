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

The `/adaptive-team-init` skill will check for this and offer to enable it automatically. To set it up manually:

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
1. Checks dependencies (Git, Claude Code version, Agent Teams enabled, git repo) — fixes what it can
2. Clones the repo to temp, copies agents/rules/skills into your `.claude/`
3. Auto-detects your tech stack (language, test framework, build system)
4. Asks targeted questions about architecture, conventions, and coverage targets
5. Writes project-specific context to `.claude/adaptive-team-context/`
6. Cleans up

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

## Skills Reference

| Skill | Scope | What it does |
|-------|-------|-------------|
| `/adaptive-team-init` | Global (install to `~/.claude/skills/`) | Full initialization — clones repo, copies files, auto-detects your stack, asks questions, writes context. One command does everything. |
| `/adaptive-team-setup` | Project | Re-runs the setup wizard to reconfigure context files after major stack changes. Use `reset` to clear and regenerate. |
| `/adaptive-team-start` | Project | Loads the PO role onto the main thread, creates a team, spawns architect + sdet. Keeps the team alive for multiple tasks in a session. |
| `/adaptive-team-plan` | Project | Design review phase — PO + architect align on approach, identify risks, write stories with acceptance criteria. Optional before implementation. |
| `/adaptive-team-implement` | Project | Full implementation — PO creates stories, architect guides, dev(s) build in worktrees, architect + sdet review, then merge. Can be used standalone or after `/adaptive-team-plan`. |

**Typical flows:**

```
Full session:   /adaptive-team-start → describe work → PO manages interactively
Plan + build:   /adaptive-team-plan <feature> → /adaptive-team-implement
Quick one-off:  /adaptive-team-implement <task>
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
    adaptive-team-init/            ← global installer (deps, files, auto-detect, configure)
    adaptive-team-setup/           ← reconfigure context after stack changes
    adaptive-team-start/           ← start a session (loads PO, creates team, spawns reviewers)
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

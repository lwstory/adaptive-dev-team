# Claude Dev Team

A reusable agent team model for Claude Code that implements a structured software development workflow with human-in-the-loop (HITL) self-healing.

## What This Is

A set of Claude Code agent definitions, rules, and skills that spin up a complete development team:

| Role | Purpose |
|------|---------|
| **Product Owner (PO)** | Orchestrates work — epics, stories, delegation, acceptance |
| **Architect** | Design guidance, code review, architectural quality gate |
| **SDET Lead** | Test strategy, test review, coverage verification |
| **Dev** | Implementation, testing, commits |

## Key Features

- **Worktree isolation** — every dev works in a git worktree, merges only after review
- **Review gates** — Architect + SDET Lead must both sign off before code is accepted
- **HITL self-healing** — operational failures become permanent improvements to role files
- **Task-scoped devs** — fresh context per task prevents bias and enables parallelism
- **Persistent reviewers** — Architect and SDET Lead maintain context across tasks

## Quick Start

1. Copy `.claude/agents/`, `.claude/rules/`, and `.claude/skills/` into your project
2. Create a `CLAUDE.md` in your project root (see `CLAUDE.md` in this repo for the template)
3. Customize the agent files with your project's tech stack and patterns
4. Use the `/dev-team` skill to spin up the team

## How It Works

```
User request
  → PO creates epics/stories/tasks
    → Architect provides design guidance
      → Dev implements in worktree
        → Architect reviews code → SATISFIED/UNSATISFIED
        → SDET Lead reviews tests → SATISFIED/UNSATISFIED
          → Both SATISFIED → PO accepts → Dev commits and merges
```

Max 2 review cycles per task. Escalates to user after 2 rejections.

## Self-Healing (HITL)

When something goes wrong — agents not responding, broken state, communication failures — the PO stops and engages the user:

1. Describes the problem clearly
2. Asks: fix now (update role files) or defer (create a task)?
3. Every operational failure becomes a permanent improvement

The team gets better over time. No silent workarounds.

## Structure

```
.claude/
  agents/
    product-owner.md   — PO role definition
    architect.md       — Architect role + review checklist
    sdet.md            — SDET Lead role + test review checklist
    dev.md             — Dev role + implementation standards
  rules/
    team-rules.md      — team composition, lifecycle, worktrees, review gates
  skills/
    dev-team/
      SKILL.md         — the skill that spins up the team
```

## Customization

The agent files ship with generic placeholders. To adapt for your project:

1. **`architect.md`** — Add your tech stack, review checklist items, architecture docs
2. **`sdet.md`** — Add your test pyramid, coverage targets, test patterns
3. **`dev.md`** — Add your code standards, test requirements, commit protocol
4. **`product-owner.md`** — Add your project context, routing rules
5. **`team-rules.md`** — Adjust team composition if needed

## License

MIT

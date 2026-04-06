---
name: dev-team
description: "Spin up a Product Owner-led dev team to implement technical work. Creates epics, stories, assigns Architect + SDET Lead + Devs."
argument-hint: "<description of the work to be done>"
---

# Dev Team

Spins up a full development team to implement technical work. The team follows the roles and processes defined in `.claude/rules/team-rules.md`.

**This skill should be used for ANY technical implementation or development work.**

## References

Read these before spawning agents — they define roles, rules, and processes:

| Document | Purpose |
|----------|---------|
| `.claude/rules/team-rules.md` | Team composition, lifecycle, worktree rules, commit protocol |
| `.claude/agents/product-owner.md` | PO role: epics, stories, delegation, acceptance |
| `.claude/agents/architect.md` | Architect role: design, code review, checklist |
| `.claude/agents/sdet.md` | SDET Lead role: test strategy, review, checklist |
| `.claude/agents/dev.md` | Dev role: implementation standards, test requirements, commit protocol |

Each agent should read their role file at the start of their work.

## Team Composition

| Role | Agent File | Lifecycle |
|------|-----------|-----------|
| Product Owner (PO) | `.claude/agents/product-owner.md` | Main thread |
| Architect | `.claude/agents/architect.md` | Persistent team member |
| SDET Lead | `.claude/agents/sdet.md` | Persistent team member |
| Dev(s) | `.claude/agents/dev.md` | Disposable per task, always in worktrees |

## Pipeline

### Step 1: Create the Team

```
TeamCreate(team_name="<descriptive-name>", description="<what the work is>")
```

### Step 2: Product Owner Creates Epics and Stories

Based on the user's request, the PO (main thread) writes:
- **Epic**: high-level goal
- **Stories**: user-facing capabilities with acceptance criteria
- **Tasks**: technical work items (TaskCreate)
- Set up task dependencies (TaskUpdate with addBlockedBy)

### Step 3: Spawn Architect for Design Guidance

Spawn the Architect as a team member (persistent, Opus model). The Architect:
- Reads relevant code
- Provides design guidance (where to make changes, what patterns to follow)
- Writes a guidance doc if the work is complex
- References `.claude/agents/architect.md` for full checklist

### Step 4: Spawn SDET Lead for Test Strategy

If the work involves new features or significant changes, spawn the SDET Lead (persistent, Opus model):
- Designs the test strategy
- Defines what tests are needed and where they go
- References `.claude/agents/sdet.md` for full checklist

### Step 5: Spawn Dev(s) for Implementation

Spawn dev agents (disposable, always with `isolation: "worktree"`):
- Each dev gets ONE task with clear acceptance criteria
- Split independent tasks across multiple devs for parallelization
- Each dev reads `.claude/agents/dev.md` for standards
- Devs report back when done — they do NOT commit

### Step 6: Review Gate

After each dev reports completion:
1. Route to Architect for code review → SATISFIED/UNSATISFIED
2. Route to SDET Lead for test review → SATISFIED/UNSATISFIED
3. If both SATISFIED → PO marks accepted → Dev commits and pushes
4. If either UNSATISFIED → route findings to dev → fix → re-review (max 2 cycles)

### Step 7: Cleanup

After all work is committed:
- Shut down all agents
- Delete the team (TeamDelete)
- Report summary to user

## When to Use This Skill

Use this skill whenever the user asks for:
- New feature implementation
- Bug fixes that require code changes
- Refactoring or restructuring
- Adding tests
- Updating infrastructure (Docker, config, DI)
- Any technical work

## When NOT to Use This Skill

- Simple config changes (one-line edits)
- Documentation-only changes (no code)
- Research or investigation tasks

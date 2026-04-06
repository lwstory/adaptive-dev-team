---
name: adaptive-team-implement
description: "Spin up the full adaptive team to implement technical work. Devs build in worktrees, reviewers sign off, then merge."
argument-hint: "<description of the work to implement>"
---

# Adaptive Team Implement

Spins up the full development team to implement technical work. Follows the roles and processes in `.claude/rules/adaptive-team-rules.md`.

**Use for any technical implementation — features, fixes, refactoring, tests.**

## References

| Document | Purpose |
|----------|---------|
| `.claude/rules/adaptive-team-rules.md` | Team composition, lifecycle, review gates |
| `.claude/agents/adaptive-team-product-owner.md` | PO role |
| `.claude/agents/adaptive-team-architect.md` | Architect role |
| `.claude/agents/adaptive-team-sdet.md` | SDET Lead role |
| `.claude/agents/adaptive-team-dev.md` | Dev role |
| `.claude/adaptive-team-context/` | Project-specific knowledge |
| `.claude/adaptive-team-learned/` | Accumulated lessons |

## Pipeline

### Step 1: Create the Team

```
TeamCreate(team_name="<descriptive-name>", description="<what the work is>")
```

### Step 2: PO Creates Stories

If not already done via `/adaptive-team-plan`:
- Epic, stories with acceptance criteria, tasks (TaskCreate)

### Step 3: Spawn Architect

Spawn adaptive-team-architect (persistent, Opus). Design guidance before devs start.

### Step 4: Spawn SDET Lead

Spawn adaptive-team-sdet (persistent, Opus). Test strategy and expectations.

### Step 5: Spawn Dev(s)

Spawn adaptive-team-dev (disposable, always `isolation: "worktree"`):
- One dev per task with clear acceptance criteria
- Split independent tasks across multiple devs
- Devs report back when done — do NOT commit

### Step 6: Review Gate

1. Route to adaptive-team-architect → SATISFIED/UNSATISFIED
2. Route to adaptive-team-sdet → SATISFIED/UNSATISFIED
3. Both SATISFIED → PO accepts → dev commits and merges
4. UNSATISFIED → route findings to dev → fix → re-review (max 2 cycles)

### Step 7: Adaptive Learning

PO checks: did anything go wrong? Did the user get what they expected?
- Route lessons to the correct file in `.claude/adaptive-team-learned/`
- See the root-cause routing table in adaptive-team-product-owner.md

### Step 8: Cleanup

- Shut down all agents
- Delete the team (TeamDelete)
- Report summary to user

## When to Use

- New features, bug fixes, refactoring, adding tests
- Infrastructure changes (Docker, config, CI/CD)
- Any technical work that needs code changes

## When NOT to Use

- Simple one-line config edits
- Documentation-only changes
- Planning/design review only (use `/adaptive-team-plan` instead)

---
name: adaptive-team-implement
description: "Spin up the full adaptive dev team to implement technical work. Devs build in worktrees, reviewers sign off, then merge."
argument-hint: "<description of the work to implement>"
---

# Adaptive Team Implement

Spins up the full development team to implement technical work. Follows the roles and processes in `.claude/rules/adaptive-team-rules.md`.

**Use for any technical implementation — features, fixes, refactoring, tests.**

## Pipeline

### Step 0: PO Startup

The main thread acts as adaptive-team-product-owner. Complete your startup sequence:
1. Read `.claude/agents/adaptive-team-product-owner.md` (your role)
2. Read `.claude/rules/adaptive-team-rules.md` (team process)
3. Read all files in `.claude/adaptive-team-context/` (project knowledge). If the directory contains only `.gitkeep`, prompt the user to run `/adaptive-team-setup` first.
4. Read all files in `.claude/adaptive-team-learned/` (accumulated lessons). If empty, skip — lessons will accumulate over time.

### Step 1: Create or Join Team

If a team already exists from a prior `/adaptive-team-plan` or `/adaptive-team-implement`, add work to it — do NOT delete and recreate.

If no team exists:
```
TeamCreate(team_name="<descriptive-name>", description="<what the work is>")
```

### Step 2: PO Creates Stories

If work was planned via `/adaptive-team-plan`, reference existing tasks. Otherwise, create them now:
- Epic, stories with acceptance criteria, tasks (TaskCreate)
- Do NOT proceed to Step 3 until tasks exist.

### Step 3: Spawn Architect

Spawn adaptive-team-architect (persistent, `model: "opus"`). Design guidance before devs start.

### Step 4: Spawn SDET

Spawn adaptive-team-sdet (persistent, `model: "opus"`). Test strategy and expectations.

### Step 5: Spawn Dev(s)

Spawn adaptive-team-dev (disposable, `model: "sonnet"` for straightforward tasks, `model: "opus"` for complex multi-file work, always `isolation: "worktree"`):
- One dev per task with clear acceptance criteria
- For parallel devs, declare file ownership boundaries in each dev's task description
- Devs report back when done — do NOT commit

### Step 6: Review Gate

Route to both reviewers. They may review in parallel:
1. adaptive-team-architect reviews code → SATISFIED/UNSATISFIED
2. adaptive-team-sdet reviews tests → SATISFIED/UNSATISFIED
3. Both SATISFIED → PO accepts → dev commits and merges
4. UNSATISFIED → route findings to dev → fix → re-review (max 2 cycles)

If post-merge tests fail: dev reverts the merge commit (`git revert`), then fixes in a new worktree. Never attempt fixes directly on main without isolation.

### Step 7: Adaptive Learning

PO checks: did anything go wrong? Did the user get what they expected?
- Diagnose root cause and route lessons to the correct file (see routing table in adaptive-team-product-owner.md)
- Project-wide policies go to `CLAUDE.md`, role-specific lessons go to `adaptive-team-learned/`

### Step 8: Report

- Report summary to user (what was delivered, test results, any open items)
- Ask the user if they have more work or if the session is complete
- Only delete the team (TeamDelete) if the user confirms all work is done

## Agent Timeout Protocol

If any agent has not reported back within 2 minutes, the PO checks its status. After 5 minutes of silence, escalate to the user.

## When to Use

- New features, bug fixes, refactoring, adding tests
- Infrastructure changes (Docker, config, CI/CD)
- Any technical work that needs code changes

## When NOT to Use

- Simple one-line config edits
- Documentation-only changes
- Planning/design review only (use `/adaptive-team-plan` instead)

# Team Development Rules

## Team Composition

The PO runs on the main thread. All other roles are Claude Team agents (subagents) that can message each other.

| Role | Lifecycle | Count | Agent File |
|------|-----------|-------|-----------|
| **Product Owner (PO)** | Main thread (persistent) | 1 | `.claude/agents/product-owner.md` |
| **Architect** | Team member (persistent for session) | 1 | `.claude/agents/architect.md` |
| **SDET Lead** | Team member (persistent for session) | 1 | `.claude/agents/sdet.md` |
| **Dev** | Team member (task-scoped, fresh per task) | 1-3 | `.claude/agents/dev.md` |

### Agent Lifecycle Rules

- **Product Owner**: Lives on the main thread. Creates the team, writes stories, orchestrates, marks acceptance. Never a subagent. Never touches code or git.
- **Architect**: Spawned as a team member. Persistent for the session — can review multiple tasks without being recreated. Messages other team members directly.
- **SDET Lead**: Same as Architect — persistent, reviews multiple tasks, messages team members.
- **Dev**: **Task-scoped**. Spawned as a dedicated agent for a single task, reports back, completes when the task is merged. A new dev agent is created for each new development task. Devs do NOT carry context between tasks.

### Why Devs Get Fresh Context Per Task

- Clean context ensures full focus on the current task without prior assumptions
- Each dev gets a targeted prompt with only the relevant task scope
- Prevents bias from prior approaches — every task gets a fresh perspective
- Multiple devs can work in parallel on independent tasks (use worktrees)

### Team Creation Pattern

```
1. PO creates team (TeamCreate)
2. PO creates epics/stories/tasks (TaskCreate)
3. PO spawns Architect for design/review (persistent)
4. PO spawns SDET Lead for test strategy/review (persistent)
5. PO spawns Dev(s) for implementation (task-scoped, one per task, always in worktrees)
6. Dev completes → Architect reviews → SDET Lead reviews → PO marks accepted
7. Dev commits and pushes after acceptance
8. Dev shut down, worktree merged
9. New Dev spawned for next task in a new worktree
10. Architect + SDET Lead stay alive across all tasks
11. Team deleted after all work is done
```

### IMPORTANT: One Team Per Session — Add Work, Don't Rebuild

**When the user requests new work while a team is active, ADD tasks and devs to the existing team.** Do NOT delete the team and create a new one. The Architect and SDET Lead are persistent — they already have codebase context and can handle multiple epics/stories within the same session.

The correct flow for new work mid-session:
1. Create new tasks (TaskCreate) on the existing team
2. Get design guidance from the existing Architect
3. Get test strategy from the existing SDET Lead
4. Spin up new task-scoped Dev(s) in worktrees

Only delete a team (TeamDelete) when ALL work for the session is complete and the user confirms they're done.

**Why this matters:**
- TeamDelete kills all team members — the persistent Architect and SDET Lead lose their context
- Recreating a team forces new agents to re-read the codebase from scratch
- The `TeamCreate` API only allows one team at a time — deleting to recreate risks broken state if the creation flow is interrupted

## Worktrees

**ALL dev agents MUST use `isolation: "worktree"` on Agent calls.** Every dev works in an isolated worktree, even single devs. This enables parallelization and prevents accidental changes to the main branch before review.

After acceptance, the dev commits in the worktree and the worktree branch is merged into the main branch.

### Why Always Worktrees
- Enables parallel dev agents without merge conflicts
- Prevents uncommitted changes from bleeding into the main branch
- Clean separation: dev works in isolation, merges only after review
- If a dev's work is rejected, the worktree can be discarded with zero impact

### When Worktrees Are NOT Needed
- Read-only agents (Architect, SDET Lead) — they review, they don't write

## Review Gate

Every dev task must pass through Architect + SDET Lead review before acceptance. No exceptions.

```
Dev completes → Architect reviews code → SDET Lead reviews tests → Both SATISFIED → PO accepts → Commit
```

Max 2 review cycles per task. Escalate to user after 2 rejections.

## Communication

- All team members can message each other via SendMessage
- Architect and SDET Lead should collaborate on findings (message each other directly)
- Devs report to PO, not directly to reviewers
- PO routes review findings to devs, not reviewers

## File Ownership

When multiple dev agents run in parallel, assign clear file ownership boundaries. No two devs should modify the same file.

## Commit and Merge Protocol

1. Dev implements in worktree, runs targeted tests during implementation, confers with Architect/SDET as needed (shift-left)
2. Dev runs FULL test suite, all pass → reports completion to PO
3. PO routes to Architect for code review → SATISFIED/UNSATISFIED
4. PO routes to SDET Lead for test review → SATISFIED/UNSATISFIED
5. If both SATISFIED → PO marks work accepted
6. **Dev commits in worktree, merges worktree branch into main, pushes**
7. **Dev runs ALL tests on main** — merge may introduce conflicts/regressions
8. **Architect + SDET Lead do final sign-off on main (base branch)** — not the worktree
9. If post-merge issues → Dev fixes on main, Architect/SDET re-verify
10. Dev agent shut down after final sign-off on main

## Role Files

All agents should read their role file before starting work:
- Product Owner: `.claude/agents/product-owner.md`
- Architect: `.claude/agents/architect.md`
- SDET Lead: `.claude/agents/sdet.md`
- Dev: `.claude/agents/dev.md`

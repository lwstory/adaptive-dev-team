# Adaptive Team Rules

## Team Composition

The PO runs on the main thread. All other roles are Claude Team agents that can message each other.

| Role | Lifecycle | Count | Agent File |
|------|-----------|-------|-----------|
| **adaptive-team-product-owner** | Main thread (persistent) | 1 | `.claude/agents/adaptive-team-product-owner.md` |
| **adaptive-team-architect** | Team member (persistent) | 1 | `.claude/agents/adaptive-team-architect.md` |
| **adaptive-team-sdet** | Team member (persistent) | 1 | `.claude/agents/adaptive-team-sdet.md` |
| **adaptive-team-dev** | Team member (task-scoped) | 1-3 | `.claude/agents/adaptive-team-dev.md` |

### Agent Lifecycle

- **adaptive-team-product-owner**: Main thread. Orchestrates, never codes.
- **adaptive-team-architect**: Persistent for the session. Reviews multiple tasks.
- **adaptive-team-sdet**: Same as architect — persistent, reviews multiple tasks.
- **adaptive-team-dev**: Task-scoped. Fresh agent per task, discarded after merge.

### Why Devs Get Fresh Context

- Full focus on current task without prior assumptions
- Prevents bias from prior approaches
- Enables parallel devs on independent tasks (worktrees)

### Team Creation Pattern

```
1. PO creates team (TeamCreate)
2. PO creates epics/stories/tasks (TaskCreate)
3. PO spawns adaptive-team-architect (persistent)
4. PO spawns adaptive-team-sdet (persistent)
5. PO spawns adaptive-team-dev(s) (task-scoped, always in worktrees)
6. Dev completes → architect reviews → sdet reviews → PO accepts
7. Dev commits, merges worktree, pushes
8. Dev shut down, new dev for next task
9. Architect + SDET stay alive across all tasks
10. Team deleted after all work is done
```

### One Team Per Session — Add Work, Don't Rebuild

**ADD tasks and devs to the existing team.** Do NOT delete and recreate. The architect and sdet are persistent — they already have context.

Only delete a team (TeamDelete) when ALL work is complete and the user confirms.

## Worktrees

**ALL dev agents MUST use `isolation: "worktree"`.** Every dev works in an isolated worktree, even single devs.

- Enables parallel devs without merge conflicts
- Prevents uncommitted changes on the main branch
- Rejected work can be discarded with zero impact
- Read-only agents (architect, sdet) do NOT need worktrees

## Review Gate

Every task: adaptive-team-architect + adaptive-team-sdet review before acceptance. No exceptions.

```
dev completes → architect reviews code → sdet reviews tests →
Both SATISFIED → PO accepts → dev commits and merges
```

Max 2 review cycles. Escalate to user after 2 rejections.

## Reviewer Context Hygiene

After 3+ review cycles in a session, the PO checks whether adaptive-team-architect and adaptive-team-sdet are still performing well. Signs of degradation: reviews that are shorter or less detailed than earlier ones, missing issues that earlier reviews caught, or repeating generic guidance instead of project-specific feedback. If quality has degraded, spawn a fresh reviewer with a summary of key findings from the session so far.

## Agent Timeout Protocol

If any agent has not reported back within 5 minutes, the PO checks its status via SendMessage and reports to the user — which agent, what it was working on, and whether it appears stuck or still progressing.

## Communication

- All members message each other via SendMessage
- Architect and sdet collaborate directly on findings
- Devs report to PO, not directly to reviewers
- PO routes review findings to devs

## File Ownership

Parallel devs get clear file ownership boundaries. PO specifies owned files/directories in each dev's task description. If a dev needs to touch a file outside their boundary, they message the PO to negotiate — never modify unowned files.

## Post-Merge Rollback

If tests fail after merging a worktree to main:
1. Dev reverts the merge commit (`git revert <merge-commit>`)
2. Dev creates a new worktree for the fix
3. Fix goes through the normal review gate
4. Never attempt fixes directly on main without worktree isolation

## Commit and Merge Protocol

1. Dev implements in worktree, runs targeted tests, confers with architect/sdet (shift-left)
2. Dev runs FULL test suite → reports completion to PO
3. PO routes to architect → SATISFIED/UNSATISFIED
4. PO routes to sdet → SATISFIED/UNSATISFIED
5. Both SATISFIED → PO accepts
6. Dev commits in worktree, merges to main, pushes
7. Dev runs ALL tests on main
8. Architect + sdet final sign-off on main (not worktree)
9. Post-merge issues → dev fixes on main, reviewers re-verify
10. Dev shut down after final sign-off

## Adaptive Learning

All lessons go to `.claude/adaptive-team-learned/`. The PO diagnoses root cause and routes to the correct file:

| File | What goes here |
|------|---------------|
| `po-lessons.md` | Requirements, scope, communication, user alignment |
| `architect-lessons.md` | Design, architecture, review criteria |
| `sdet-lessons.md` | Test strategy, coverage, quality patterns |
| `dev-lessons.md` | Implementation patterns, build issues, coding standards |
| `team-lessons.md` | Process, coordination, agent communication |

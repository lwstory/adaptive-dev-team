---
name: adaptive-team-implement
description: "Spin up the full adaptive dev team to implement technical work. Reviewers brief the dev, dev builds in a worktree, reviewers sign off, then merge."
argument-hint: "<description of the work to implement>"
---

# Adaptive Team Implement

Full development pipeline with PM context discipline, shift-left briefings, architect-chosen dev model, and direct reviewer→dev feedback.

## Pipeline

### Step 0: PM Recovery / Grounding

PM reads, in order:
1. `.claude/rules/adaptive-team-rules.md`
2. `.claude/adaptive-team-state/current-session.md` (initialize from canonical structure in rules if missing)
3. `~/.claude/pm-user-lessons.md` (user-global)
4. `.claude/adaptive-team-learned/pm-lessons.md` (project-PM)
5. `TaskList`

### Step 0.5: Ambiguity Check (P2 — auto-consult)

Before spawning anything: can the user's acceptance criteria be expressed in **one declarative sentence**? If not, route to `/adaptive-team-consult` first. Ambiguity is a signal; consult is cheap; bad implementation isn't. Do not bypass.

### Step 1: Create or Join Team

If a team exists, join it — do not recreate. Otherwise `TeamCreate`. Update `current-session.md`.

### Step 2: Break Down Work

PM creates epic / stories / tasks via `TaskCreate`. Each task has clear acceptance criteria and file-ownership boundaries.

### Step 3: Spawn Persistent Reviewers by Signal

Always spawn: `architect`, `sdet` (code is being written).
Add on signal (per Consult Mode rubric): `llm-expert`, `database`.

Use deterministic names. Each reviewer completes its startup sequence.

### Step 4: Shift-Left Briefings + Dev Model Selection

For each task, PM asks each relevant reviewer to SendMessage a plain-English briefing. Briefings are:
- Principle-first, task-specific
- Not code dumps

PM **also asks the `architect`** which model the dev should use for this task (Sonnet for straightforward, Opus for complex multi-file). Architect replies with the model choice.

### Step 5: Spawn Dev(s)

For each task: `Agent` with `name: "dev"` (or `dev-<n>` for parallel), `isolation: "worktree"`, and the `model` value provided by the architect.

The dev:
- Reads role file, rules, dev-lessons, briefings, and the specific files in the task description
- Does **not** read the whole codebase
- Implements; runs targeted tests; may confer with reviewers via SendMessage
- Runs the full suite; reports completion to PM (not to reviewers)

### Step 6: Review Gate

PM routes to each relevant reviewer. Each reviewer:
- Reads the dev's completion report and code as needed
- SendMessages required changes **directly to the dev**
- Writes full findings to `.claude/adaptive-team-reviews/<story>/<task>/<reviewer>-<cycle>.md`
- Returns a verdict to PM in the strict 3-line format

If UNSATISFIED: dev may surface concerns to the reviewer (information, not pushback). Reviewer weighs input and decides. Dev implements feedback. Re-review. Max 2 cycles → escalate to user.

On UNSATISFIED, reviewer self-reflects: *did I brief adequately?* Proposes a lesson (→ own file if inadequate briefing, → `dev-lessons.md` if dev side) to PM → user for approval.

### Step 7: Merge

All SATISFIED → PM accepts → dev commits in worktree → merges to main → runs full tests on main → final reviewer sign-off on main → dev shut down. Update `current-session.md`.

### Step 8: Post-Merge Rollback (if needed)

Post-merge failures: dev reverts the merge commit, creates a new worktree for the fix. Back through the gate. Never fix directly on main.

### Step 9: Retrospective

PM checks with user: did the result match what was wanted? Any issues → route to `/adaptive-team-learning-moment`.

Delete the team only when user confirms all work is complete.

## Agent Timeout

Any team member silent >5 minutes → PM checks via SendMessage, reports status to user.

## When to Use

Any technical work that requires code changes.

## When NOT to Use

- Design / approach discussion only → `/adaptive-team-consult`
- Capture a lesson → `/adaptive-team-learning-moment`

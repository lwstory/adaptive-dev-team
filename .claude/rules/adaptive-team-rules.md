# Adaptive Team Rules

## Team Composition

The PM runs on the main thread. All other roles are Claude Team members that message each other.

| Role | Lifecycle | Count | Deterministic Name |
|------|-----------|-------|--------------------|
| **product-manager** | Main thread (persistent) | 1 | n/a (main thread) |
| **architect** | Team member (persistent) | 1 | `architect` |
| **sdet** | Team member (persistent) | 1 | `sdet` |
| **llm-expert** | On-demand, persistent once spawned | 0-1 | `llm-expert` |
| **database** | On-demand, persistent once spawned | 0-1 | `database` |
| **curious** | On-demand (ideation), persistent once spawned | 0-1 | `curious` |
| **dev** | Task-scoped (fresh per task) | 0-3 | `dev` or `dev-<n>` |

**Deterministic names are mandatory.** PM addresses teammates by role name via SendMessage. Names survive compaction; IDs do not. Spawning with any other name is forbidden.

Team members, not "sub-agents." Use "team member" / "teammate" everywhere.

## Identity Block Contract (L2)

Every role file's **first ~30 lines must be self-sufficient** — if the rest of the file is unavailable (compacted, partial read, harness truncation), the first 30 lines must carry:

1. Role statement (one sentence — what this agent is accountable for)
2. Tool constraints (only where restricted — primarily PM)
3. Verdict format (for reviewers: architect, sdet, llm-expert, database)
4. Startup pointers (which files to read; sizes listed per §Exhaustive Startup)

Role files violating this contract are malformed. Drift-resistance is a contract, not a hope.

## Exhaustive Startup (L8)

Each role file's `## Startup` section lists **every file read at startup** with an **approximate size** (`~X KB`). Sizes are derivable by inspection, not advertised numbers. Two reasons:
- A reviewer can sanity-check its own context budget at startup rather than discover overflow mid-turn.
- A PM planning a new task can estimate cold-start cost before spawning.

When a startup read is added or a file grows materially, update the size hint in the role file.

## PM Context Discipline

The PM must not pull review findings, source code, or build output into main-thread context. Breaking this causes compaction cascades that wreck session health.

Enforced by the PM's **tool palette** (see `.claude/agents/adaptive-team-product-manager.md`):
- PM may Read only: this rules file, `current-session.md`, `pm-lessons.md`, `~/.claude/pm-user-lessons.md`, its own role file
- PM may Write/Edit only under `.claude/adaptive-team-state/`, `.claude/adaptive-team-reviews/` (its own notes; never reviewers' DETAILS files), `.claude/adaptive-team-learned/` (for user-approved lessons)
- PM may NOT Read, Grep, Bash, or Edit source code
- PM communicates via SendMessage and tracks via Task tools

If PM needs to know code state, it asks the architect.

## P9 — PM Blocker Restatement

When PM relays an UNSATISFIED to the user, PM restates the blocker in **≤1 sentence using only the REASON line**. PM does NOT open DETAILS. The restatement adds comprehension without violating discipline. A verbatim REASON dump is acceptable; a findings paraphrase is not.

## Session State

`.claude/adaptive-team-state/current-session.md` is PM's durable memory. PM updates it on every material event (team member spawned/shutdown, task phase change, verdict received, decision made). PM re-reads it on recovery.

### Canonical Session State Structure

```
# Active Session

## Team
- architect: <not spawned | alive | standing down>
- sdet: <not spawned | alive | standing down>
- llm-expert: <not spawned | alive | standing down>
- database: <not spawned | alive | standing down>
- curious: <not spawned | alive | standing down>
- dev: <none | alive (task T<id>)>

## In-flight
Task: <id> — <short title>
Phase: <idle | briefing | dev-working | under-review | accepted | merging>
Last verdict: <reviewer>: <SATISFIED | UNSATISFIED | BLOCKED>
Awaiting: <who, for what>

## Decisions this session
- [signal: "<topic phrase>" → spawned: <name>]
- <timestamp>: <decision>

## Recent Activity (rolling tail — last 20 events)
- <timestamp> <event>
```

**Phase values are enumerated above** — no free-text inventions. If state ambiguous on recovery, reconcile against `TaskList` + observable team membership; trust `TaskList` + roster; rewrite state; note the divergence in Decisions.

## P5 — Pending PM Lessons (staging)

`.claude/adaptive-team-state/pending-pm-lessons.md` holds in-flight PM observations before user approval. PM writes proposals there at the moment of noticing. At session boundaries (or on `/adaptive-team-learning-moment`), PM reviews pending proposals with user. Approved text promotes to the target file (`~/.claude/pm-user-lessons.md` for user-interaction; `pm-lessons.md` for project-PM); the staging entry is cleared.

This is the durable-lesson-proposal pattern (closes the database-audit "no transaction boundary for propose→approve→write" concern).

## PM Recovery Protocol

If PM senses drift (forgets a team member, unsure of current task, unclear on a rule):
1. Read `.claude/rules/adaptive-team-rules.md`
2. Read `.claude/adaptive-team-state/current-session.md`
3. Read `~/.claude/pm-user-lessons.md`
4. Read `.claude/adaptive-team-learned/pm-lessons.md`
5. `TaskList`

Only then decide next action. No guessing.

## Reviewer Protocol

All reviewers (`architect`, `sdet`, `llm-expert`, `database`) follow this lifecycle for every dev task:

### 1. Shift-Left Briefing (before dev starts)

Before the dev writes a line of code, PM asks each relevant reviewer to SendMessage a **briefing** to the dev. The briefing:
- Is plain English, principle-first
- Names concrete pitfalls for this specific task
- Does not dump code (one short illustrative snippet is OK)
- Tells the dev to read their lessons file first
- Ends with the architect's red-team clause label (`**Careless-dev failure mode:**`) and a `## Gotchas` section (architect-owned briefings; others optional)

PM does not ingest briefing content.

**Architect also specifies the dev's model** for this task (Sonnet for straightforward, Opus for complex multi-file work). PM spawns the dev with the architect-chosen model.

### 2. Monitor During Work

Reviewers stay available. Dev may confer directly with any reviewer (shift-left, encouraged). Dev does not request sign-off from reviewers — PM owns acceptance routing.

### 3. Review on Completion

Dev reports completion to PM → PM asks each relevant reviewer to review. Reviewer:
- Reads the dev's completion report and code/test artifacts as needed
- Writes full findings to `.claude/adaptive-team-reviews/<story-id>/<task-id>/<reviewer>-<cycle>.md`
- SendMessages required changes directly to the dev (not to PM)
- Returns a **verdict** to PM

### 4. Verdict Format (strict)

Reviewer → PM message must be exactly:

```
VERDICT: SATISFIED | UNSATISFIED | BLOCKED
REASON: <one line, <=120 chars>
DETAILS: <path to findings file>
```

Anything longer leaks into PM context. If a reviewer needs to say more, it goes in the DETAILS file. PM treats any message from a reviewer that doesn't match this format as malformed and replies "resend in strict format" without scanning content.

### 5. Reviewer Self-Reflection (on UNSATISFIED)

Before proposing a dev lesson, the reviewer asks itself: *did I brief this adequately?*
- If no → lesson goes to the reviewer's own lessons file
- If yes → lesson goes to `dev-lessons.md`
- Genuine judgment call → escalate to user via PM

**The dev never self-assesses.** The reviewer owns the diagnosis. Devs may surface concerns (information for the reviewer to weigh), but don't claim correctness.

**Before issuing a scope-drift finding**, the reviewer must verify attribution via `git blame` or `git log --oneline -- <file>` on the specific lines. Session memory is not sufficient — false-positive scope-drift blocks devs on non-issues.

### 6. Lesson Proposal Routing

Reviewer drafts the lesson in plain English, role-file style, with a concrete example. SendMessages the proposal to PM. PM forwards it **verbatim** to the user. User approves, edits, or rejects. Only after user approval does PM write the approved text to the target file.

**No file under `.claude/adaptive-team-learned/`, `~/.claude/pm-user-lessons.md`, or `CLAUDE.md` is written without explicit user approval.**

## P3 — Reviewer Freshness Audit

After every **3rd task** in a session, PM asks each persistent reviewer (one sentence): "name your most recent concrete finding." Signs of degradation:
- Short or generic answer → spawn a fresh reviewer with a summary handoff
- "I don't remember" → same
- Verbose / rambling → reviewer has absorbed too much review context; spawn fresh

Observable, deterministic trigger — not PM-intuition-based.

## No Auto-Promotion to CLAUDE.md

`CLAUDE.md` receives additions only when a specific issue *at that moment* warrants project-wide policy, user-decided at that moment. **Never promote lessons from learning files into CLAUDE.md as a cleanup step.** CLAUDE.md stays lean.

## Consult Mode — Reviewer Selection

When `/adaptive-team-consult` fires, PM selects reviewers by topic signal:

| Signal | Spawn |
|--------|-------|
| Schema, query, migration, index, graph | `database` |
| Prompt, tool use, model choice, agent design, eval | `llm-expert` |
| Auth, permissions, authz, secrets, trust boundary | `architect` (owns permission lens) |
| Test strategy, coverage, flake | `sdet` |
| Architecture, coherence | `architect` |
| Ideation, brainstorm, "what could we do differently", outside-the-box | `curious` |
| Ambiguous / unclear | `architect` as default |

Default minimum: `architect` always. Add others by signal, not by habit. Log decision: `[signal: "<phrase>" → spawned: <name>]` in `current-session.md`.

## Worktrees

All dev agents MUST use `isolation: "worktree"`. Every dev, every task, even solo. Read-only reviewers (architect, sdet, llm-expert, database, curious) do not use worktrees.

**Path discipline in the worktree:** devs MUST use paths relative to `$WORKTREE_ROOT`, never absolute paths to the main repo. Spawning with `isolation: "worktree"` doesn't prevent a dev from writing to absolute main-repo paths — that's role discipline, enforced via the dev role file.

## Review Gate

Every implementation task: all relevant reviewers must return SATISFIED before PM accepts. Max 2 review cycles. If still UNSATISFIED after 2, PM escalates to user.

## Dev Startup Checklist

A fresh dev on spawn reads, in order:
1. Its role file (`adaptive-team-dev.md`)
2. `adaptive-team-rules.md`
3. `dev-lessons.md`
4. Briefings received via SendMessage
5. Files listed in the task description — **not the whole codebase**
6. Any `<reviewer>-<cycle>.md` in the task's review dir (only for re-dos)
7. **Pre-flight echo (B1) — MANDATORY gate.** SendMessage the briefing author 3 bullets: "My understanding is [bullet 1]; [bullet 2]; [bullet 3]." Wait for "confirmed" or correction. The dev MUST NOT write any code before confirmation lands.
8. **Unknown-unknowns ask (B8) — MANDATORY gate.** Immediately after B1 confirmation, SendMessage the briefing author one question — the single thing the dev is most uncertain about. If genuinely uncertain about nothing, ask: "What am I most likely underestimating?" Wait for the answer. The dev MUST NOT write any code before the answer lands.

If orientation is needed, ask the architect. Do not explore.

## Commit and Merge Protocol

1. Dev implements in worktree; runs targeted tests; confers with reviewers (shift-left)
2. Dev runs full test suite; reports completion to PM
3. PM routes to relevant reviewers; verdicts return
4. All SATISFIED → PM accepts
5. Dev commits in worktree, merges to main, pushes
6. Dev runs full tests on main
7. Reviewers final sign-off on main
8. Dev shut down

Post-merge failure: dev reverts the merge commit, creates a new worktree for the fix. Never fix directly on main.

## Agent Timeout Protocol

If any team member has not reported back within 5 minutes, PM checks via SendMessage and reports to user — which member, what it was doing, stuck vs progressing.

## One Team Per Session

Add work to the existing team. Do not delete and recreate. Only `TeamDelete` when user confirms all work complete.

## Learning File Size Governance

When any `.claude/adaptive-team-learned/*.md` exceeds **500 lines**, its owning role proposes a consolidation (merge duplicates, drop stale). **No promotion to CLAUDE.md as part of cleanup.** User approves.

## Communication

- All teammates use SendMessage **by name**
- Reviewers SendMessage findings directly to the dev
- PM receives only verdicts (strict format above) plus lesson proposals
- Dev reports completion to PM, not to reviewers
- Reviewer confirmations of dev pre-flight echoes go **directly to the dev**, not to PM (PM sees verdicts, not echo acks)

## File Ownership

Parallel devs each get explicit file/dir ownership in the task description. If a dev needs to touch a file outside its boundary, it asks PM to negotiate — never modifies unowned files.

# HITL Self-Healing Process

## Overview

Human-in-the-Loop (HITL) self-healing is the principle that **every operational failure becomes a candidate for a permanent improvement — and no improvement lands without the user saying yes.** Lessons accumulate in role-specific files under `.claude/adaptive-team-learned/` (project) and `~/.claude/pm-user-lessons.md` (user-global for communication style). Agent role files stay thin and stable; lessons carry the project-specific deltas.

Two non-negotiables:

1. **The user approves every lesson.** No file under `.claude/adaptive-team-learned/`, `~/.claude/pm-user-lessons.md`, or `CLAUDE.md` is written without explicit user approval of the proposed text. PM forwards proposals **verbatim**; the user may approve, edit, or reject.
2. **CLAUDE.md is never populated by promotion.** Lessons don't get "escalated" from learning files to CLAUDE.md as a cleanup step. CLAUDE.md gets additions only when a specific issue *at that moment* warrants project-wide policy, decided by the user at that moment.

## How It Works

1. **Something goes wrong** — dev work fails review, test gap discovered, user dissatisfied, process friction surfaces
2. **Reviewer self-reflects first** — when the reviewer issues an UNSATISFIED verdict, before proposing a dev lesson they ask themselves: *did I brief this adequately?* Answer determines routing (see below).
3. **Reviewer drafts the lesson** — plain English, role-file style, with a concrete example. SendMessages the proposal to PM.
4. **PM forwards verbatim to user** — no paraphrasing, no summarization.
5. **User approves, edits, or rejects** — approved text (and only approved text) is written by PM.
6. **Written to the routed file** — PM appends the approved text using the standard format.

## Reviewer Self-Reflection (the core loop)

When a reviewer issues an UNSATISFIED:

- **Did I brief adequately?** If not → lesson goes to the reviewer's **own** lessons file (`architect-lessons.md`, `sdet-lessons.md`, etc.). This is the key anti-pattern prevention: without self-reflection, teams accumulate a pile of "dev should have known" lessons while reviewer blind spots persist.
- **Dev missed something they should have known** → lesson goes to `dev-lessons.md`.
- **Genuine judgment call** → escalate to user via PM; user decides where it belongs (or whether it belongs anywhere).

**Dev never self-assesses right vs wrong.** Dev may surface concerns to the reviewer as *information* (context, constraints, alternatives considered and why rejected), but does not claim correctness. The reviewer owns the diagnosis.

## Root-Cause Routing

| Root cause | Lesson File | Examples |
|-----------|-------------|----------|
| **User communication, style, question timing, tone** | **`~/.claude/pm-user-lessons.md`** (cross-project) | User repeated themselves → PM missed something; user corrected tone → PM should adjust; user answered something inferrable from state → PM should check state next time |
| **Project-PM process** | `pm-lessons.md` (project) | Scope read mis-sized, reviewer-selection miss, coordination cadence on this project |
| **Design/architecture** | `architect-lessons.md` | Abstraction leak, coherence gap, permissions/authz blind spot, attribution-verification miss |
| **Test strategy/coverage** | `sdet-lessons.md` | Coverage-hole not caught, pyramid layer wrong, flake pattern |
| **LLM-specific** | `llm-lessons.md` | Unbounded context, missing structured output contract, prompt injection surface |
| **Data-layer** | `database-lessons.md` | Missing tenant scope in index, migration locked a hot table, N+1 missed |
| **Implementation pattern** | `dev-lessons.md` | Coding convention miss, build hygiene |
| **Cross-cutting process** | `team-lessons.md` | Communication breakdown, review-gate failure, coordination |
| **Ideation pattern** | `curious-lessons.md` | What kinds of ideas land vs don't; framing that eases specialist vetting |

**No blanket-dumping.** A bug that slipped testing is an SDET lesson, not a PM lesson. A design that didn't scale is an architect lesson. Accurate diagnosis determines the right agent learns.

## User-Global vs Project-PM Split

PM observations split two ways:

- **`~/.claude/pm-user-lessons.md`** — how to *serve and communicate with* this user. Cross-project; applies everywhere.
- **`.claude/adaptive-team-learned/pm-lessons.md`** — how to *run the process* on this specific project.

Test: *does this observation describe the user, or describe the project?* User → global. Project → local. When in doubt, ask at approval time.

## Staging — the durable proposal

Lessons in-flight (before approval) live at `.claude/adaptive-team-state/pending-pm-lessons.md`. PM writes observations there at the moment of noticing — not "I'll remember this and propose later" (which loses the observation to compaction). At natural boundaries (session end, `/adaptive-team-learning-moment` invocation), PM reviews pending entries with the user. Approved entries promote; rejected entries are deleted from staging.

This closes a correctness gap: without durable staging, an approved proposal can be lost if PM context compacts between approval and write.

## Lesson Format

Every lesson entry follows this structure:

```markdown
## YYYY-MM-DD: Short title

Rule or fact in one paragraph.

**Why:** Reason / past incident.
**How to apply:** When/where this kicks in.
```

Example from `architect-lessons.md`:
```markdown
## 2026-05-01: Verify attribution via git blame before scope-drift findings

Before issuing any "scope drift" or "out-of-boundary edit" finding, run
`git blame` or `git log --oneline -- <file>` on the specific lines.

**Why:** Session memory conflates setup-phase edits with task-phase edits. A
diff pattern that looks like scope creep may predate the task's branch point.
Going off intuition produces false positives that block devs on non-issues.

**How to apply:** Any scope-drift finding. If blame shows the lines predate
the task branch, the finding does not apply — do not write it.
```

## Anti-Patterns

- **Writing lessons without user approval** — violates the non-negotiable. Lessons are proposals until approved.
- **Paraphrasing lesson proposals on the way to the user** — PM forwards verbatim.
- **Routing everything to `pm-lessons.md` or `team-lessons.md`** — the "junk drawer" anti-pattern; defeats routing.
- **Promoting lessons to `CLAUDE.md`** — CLAUDE.md is not downstream of learning files. Additions are decided at the moment of the specific issue.
- **Silently working around a problem without surfacing it** — every issue gets surfaced, even if the fix is deferred.
- **Skipping reviewer self-reflection** — defaulting every UNSATISFIED to a dev lesson is exactly how reviewer blind spots become permanent.

## What Triggers a Lesson Proposal

| Trigger | Where to look |
|---------|---------------|
| UNSATISFIED review cycle | Reviewer self-reflection first; then own-file or `dev-lessons.md` |
| User rejected a completed feature | Project-PM or user-global (split by *was it process or communication?*) |
| Build failed after merge | `dev-lessons.md` (if implementation) or `team-lessons.md` (if process) |
| Reviewer issued a wrong finding (e.g. false-positive scope drift) | Reviewer's own lessons file |
| User repeated themselves | `~/.claude/pm-user-lessons.md` — PM missed something the first time |
| Pattern repeats across sessions | Whichever file matches the root cause; codify after second occurrence |

## The Two-Strike Rule

- **First occurrence:** reviewer or PM notes the issue and proposes the fix.
- **Second occurrence:** the lesson MUST be proposed before continuing. User approval still required.

Repeat-without-codification is the silent failure the whole loop is designed to prevent.

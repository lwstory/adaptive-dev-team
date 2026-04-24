---
name: adaptive-team-learning-moment
description: "Capture lessons from an issue. Any reviewer or PM may propose; PM routes through the user for approval; approved text lands in the right file."
argument-hint: "[optional: brief description of what went wrong]"
---

# Adaptive Team Learning Moment

Any reviewer (or the PM itself) can propose a lesson. **Every lesson routes through the user for explicit approval before being written.**

## Pipeline

### Step 0: PM Recovery / Grounding

PM reads, in order:
1. `.claude/rules/adaptive-team-rules.md`
2. `.claude/adaptive-team-state/current-session.md`
3. `.claude/adaptive-team-state/pending-pm-lessons.md` (staging — any already-drafted PM observations to promote?)
4. `~/.claude/pm-user-lessons.md` (user-global)
5. `.claude/adaptive-team-learned/pm-lessons.md` (project-PM)
6. `TaskList`

### Step 1: Identify the Issue

PM synthesizes from conversation what went wrong. If ambiguous, PM asks user one focused question — not an interrogation.

### Step 2: Invite Reviewer Diagnosis

If the issue touches a reviewer's lens, PM asks that reviewer (already spawned, or spawn now) for its diagnosis. Reviewer self-reflects: *was this my briefing gap, or a dev-side miss, or a design/test blind spot I didn't catch?*

Reviewer drafts the lesson in plain English, role-file style, with a concrete example. Sends to PM.

### Step 3: PM Proposes to User

PM forwards each proposed lesson to the user **verbatim**, indicating target file:

```
Proposed lesson for <file>:
<verbatim reviewer or PM draft>
```

PM may also propose its own pm-lessons if the issue was user-communication / process / question-timing / pattern-recognition failure.

### Step 4: User Approves / Edits / Rejects

User decides per lesson. PM writes only approved text. No shortcuts.

### Step 5: Write

PM appends approved text to the target file using the standard format:

```markdown
## YYYY-MM-DD: Short title

Rule or fact in one paragraph.

**Why:** Reason / past incident.
**How to apply:** When/where this kicks in.
```

Routing table:

| Root cause | File |
|------------|------|
| **User communication, style, question timing, tone** | **`~/.claude/pm-user-lessons.md`** (global — cross-project) |
| Project-PM process, coordination, scope-reading for this project | `.claude/adaptive-team-learned/pm-lessons.md` |
| Design, permissions/authz, coherence | `architect-lessons.md` |
| Test strategy, coverage | `sdet-lessons.md` |
| Prompt, model selection, agent design, LLM failure modes | `llm-lessons.md` |
| Schema, query, migration, graph modeling | `database-lessons.md` |
| Implementation patterns | `dev-lessons.md` |
| Cross-cutting process | `team-lessons.md` |
| Ideation patterns (what lands, what doesn't) | `curious-lessons.md` |
| Project-wide policy (only when the moment warrants) | `CLAUDE.md` |

**User-global vs project-PM split:** If the lesson is about *how to communicate with this user* (their style, preferences, friction patterns), route to `~/.claude/pm-user-lessons.md`. If it's about *how to run the process on this project* (scope reading, coordination cadence), route to project-level `pm-lessons.md`. When in doubt, ask the user at approval time.

**CLAUDE.md receives additions only when a specific issue at that moment warrants project-wide policy.** Never as cleanup promotion from learning files.

## When to Use

- Something went wrong and you want to capture the lesson now
- After a failed delivery, user dissatisfaction, or process breakdown
- Anytime — works standalone or with an active team

## When NOT to Use

- Routine retrospective already covered in `/adaptive-team-implement` Step 9
- The lesson already exists — check first to avoid duplicates

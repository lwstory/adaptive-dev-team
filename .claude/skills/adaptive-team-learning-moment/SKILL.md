---
name: adaptive-team-learning-moment
description: "Capture lessons from an issue — PO diagnoses root cause from conversation context, proposes lessons, user accepts."
argument-hint: "[optional: brief description of what went wrong]"
---

# Adaptive Team Learning Moment

The PO reviews the current conversation to understand what went wrong, diagnoses root cause, and proposes lessons to add to the appropriate `adaptive-team-learned/` files.

**Use this when something went wrong and you want to capture the lesson before moving on.**

## Pipeline

### Step 1: PO Startup (if not already active)

If not already in PO mode, read:
1. `.claude/agents/adaptive-team-product-owner.md` (role identity)
2. `.claude/rules/adaptive-team-rules.md` (team process)
3. All files in `.claude/adaptive-team-context/` (project knowledge)
4. All files in `.claude/adaptive-team-learned/` (existing lessons)

If already in PO mode from `/adaptive-team-start`, skip this step.

### Step 2: Diagnose

Review the conversation history to understand what went wrong. Look for:
- What was the user trying to accomplish?
- What actually happened?
- Where did expectations diverge from results?
- Was this a requirements, design, testing, implementation, or process issue?

If the argument provides a description, use that as a starting point.

If the root cause is unclear from conversation context, ask the user **one focused question** — do not interrogate. Example: "It looks like the auth middleware failed because it wasn't tested against the new schema. Is that right, or was the issue somewhere else?"

If a team is active (architect/sdet are running), consult them for their perspective before proposing lessons. The architect may spot a design gap the PO missed; the sdet may identify a testing blind spot. Incorporate their input into the diagnosis.

### Step 3: Propose Lessons

Present lessons to the user using this format:

```
Learning moment: <short title>

Trigger: <what happened>
Root cause: <why it happened>

Proposed lessons:

1. → <file> — <lesson summary>
   <full lesson text in the standard format>

2. → <file> — <lesson summary>
   <full lesson text in the standard format>
```

Route each lesson to the correct file using the root-cause routing table:

| Root Cause | File |
|-----------|------|
| Requirements/scope | `po-lessons.md` |
| Design/architecture | `architect-lessons.md` |
| Test coverage/quality | `sdet-lessons.md` |
| Implementation | `dev-lessons.md` |
| Team process | `team-lessons.md` |
| Project-wide policy | `CLAUDE.md` |

A single issue may produce lessons for multiple files if multiple root causes contributed.

### Step 4: User Accepts

Wait for the user to accept, modify, or reject each proposed lesson. Then write accepted lessons to the corresponding files using the standard format:

```markdown
## YYYY-MM-DD: Short description

**Trigger:** What happened
**Root cause:** Why it happened
**Lesson:** What to do differently
```

Confirm what was written and where.

## When to Use

- An issue just surfaced and you want to capture the lesson now
- After a failed deploy, broken test, user dissatisfaction, or process breakdown
- Anytime during or outside a team session — works standalone or with an active team

## When NOT to Use

- Routine post-delivery retrospective — that's already built into `/adaptive-team-implement` Step 7
- The issue is already captured in `adaptive-team-learned/` — check existing lessons first to avoid duplicates

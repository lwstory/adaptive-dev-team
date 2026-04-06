---
name: adaptive-team-plan
description: "Design review — PO and Architect align on approach before implementation. Creates epics, stories, and design guidance."
argument-hint: "<description of the work to plan>"
---

# Adaptive Team Plan

Runs the planning phase of a development task. The PO (main thread) and adaptive-team-architect align on approach, identify risks, and produce stories with acceptance criteria before any implementation starts.

**Use this when you want to plan and review an approach before building.**

## Pipeline

### Step 1: PO Reads Context

The PO (main thread) reads:
- `.claude/rules/adaptive-team-rules.md`
- All files in `.claude/adaptive-team-context/`
- All files in `.claude/adaptive-team-learned/`

### Step 2: Break Down the Work

PO creates:
- **Epic**: high-level goal (1-2 sentences)
- **Stories**: user-facing capabilities with acceptance criteria
- **Tasks**: technical work items (TaskCreate)

### Step 3: Spawn Architect for Design Review

Spawn adaptive-team-architect (persistent, Opus model). The architect:
- Reads the codebase as needed
- Provides design guidance (approach, interfaces, file placement, pitfalls)
- Identifies risks and dependencies

### Step 4: Spawn SDET for Test Strategy

If the work involves new features or significant changes, spawn adaptive-team-sdet (persistent, Opus):
- Designs test strategy (which layers, what coverage)
- Defines test expectations for the dev

### Step 5: Present Plan to User

Show the user:
- Epics/stories with acceptance criteria
- Architect's design guidance
- SDET's test strategy
- Any risks or open questions

Ask if they want to adjust before implementation.

## When to Use

- Complex features that benefit from design discussion before coding
- Work where the user wants to review the approach first
- Multi-story epics that need to be broken down
- Skip this and go straight to `/adaptive-team-implement` for straightforward tasks

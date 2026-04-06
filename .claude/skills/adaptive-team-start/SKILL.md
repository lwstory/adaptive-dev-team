---
name: adaptive-team-start
description: "Start the Adaptive Dev Team — loads the PO role onto the main thread and creates/joins a team, ready for work."
argument-hint: "[optional: team name]"
---

# Adaptive Team Start

Sets up the main thread as the adaptive-team-product-owner and creates (or joins) a team. After this, you can assign work directly — no need to invoke `/adaptive-team-implement` for each task.

**Use this when you want to stay in PO mode for a session and manage multiple tasks interactively.**

## Pipeline

### Step 1: PO Startup

You are now **adaptive-team-product-owner**. Complete the startup sequence:

1. Read `.claude/agents/adaptive-team-product-owner.md` (your role identity)
2. Read `.claude/rules/adaptive-team-rules.md` (team process)
3. Read all files in `.claude/adaptive-team-context/` — if the directory contains only `.gitkeep`, prompt the user to run `/adaptive-team-setup` first
4. Read all files in `.claude/adaptive-team-learned/` — if empty, skip (lessons accumulate over time)

### Step 2: Create or Join Team

If a team already exists, join it — do NOT recreate.

If no team exists:
```
TeamCreate(team_name="<name from argument, or ask user>", description="Adaptive Dev Team session")
```

### Step 3: Spawn Persistent Reviewers

Spawn both reviewers so they're ready when work arrives:

1. Spawn **adaptive-team-architect** (persistent, `model: "opus"`)
2. Spawn **adaptive-team-sdet** (persistent, `model: "opus"`)

Both complete their startup sequences (read role file, rules, context, learned).

### Step 4: Report Ready

```
Adaptive Dev Team is online.

Team: <team name>
Roles active:
  - adaptive-team-product-owner (you, main thread)
  - adaptive-team-architect (persistent, ready)
  - adaptive-team-sdet (persistent, ready)

Ready to accept work. Describe what you need built, or use:
  /adaptive-team-plan <feature>     — design review before building
  /adaptive-team-implement <task>   — spin up dev(s) to build
  Or just describe the work and I'll create stories and assign devs.
```

## When to Use

- Starting a development session where you'll give multiple tasks
- When you want the PO to manage work interactively instead of via a single skill invocation
- When you want architect and sdet pre-loaded and ready

## When NOT to Use

- Quick one-off tasks — use `/adaptive-team-implement` directly
- Setup/configuration — use `/adaptive-team-setup`

# CLAUDE.md

## Project Overview

This repository contains the Adaptive Team — a self-healing Claude Code agent team for structured software development with human-in-the-loop learning.

## For Implementation Work

Use `/adaptive-team-implement` to spin up the full team. Use `/adaptive-team-plan` to align on approach first.

## Agent Architecture

- **Agent files** (`.claude/agents/adaptive-team-*.md`) define role identity only (~100 lines each)
- **Context files** (`.claude/adaptive-team-context/`) hold project-specific knowledge — populated by `/adaptive-team-setup`
- **Learned files** (`.claude/adaptive-team-learned/`) hold accumulated lessons from HITL self-healing
- **Rules** (`.claude/rules/adaptive-team-rules.md`) define team process

Agents read their role file, then load context and learned files at startup. This separation keeps agent definitions thin and reusable while project knowledge and lessons are isolated and replaceable.

## Key Principles

- **Adaptive learning** — every failure becomes a lesson in the right `adaptive-team-learned/` file
- **Root-cause routing** — the PO diagnoses whether a problem is requirements, architecture, testing, or process and routes the lesson accordingly
- **One team per session** — add work to existing teams, don't rebuild
- **Task-scoped devs** — fresh agent per task, no context carry-over
- **Persistent reviewers** — architect and sdet maintain context across tasks
- **Worktree isolation** — all devs work in worktrees, merge only after review

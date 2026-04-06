# CLAUDE.md

## Project Overview

This repository contains the Adaptive Dev Team — a self-healing Claude Code agent team for structured software development with human-in-the-loop learning.

## Skills

| Skill | What it does |
|-------|-------------|
| `/adaptive-team-init` | Install the framework into a project (global skill — pulls repo, copies files) |
| `/adaptive-team-setup` | Interactive wizard — discovers your codebase, populates `adaptive-team-context/` |
| `/adaptive-team-start` | Start a session — loads PO role, creates team, spawns architect + sdet |
| `/adaptive-team-plan` | Design review — PO + architect align on approach before implementation |
| `/adaptive-team-implement` | Full team implementation — devs build in worktrees, reviewers sign off, merge |

**For a full session**, use `/adaptive-team-start` — it loads the PO and keeps the team alive for multiple tasks. For a quick one-off, use `/adaptive-team-implement` directly.

## Agent Architecture

- **Agent files** (`.claude/agents/adaptive-team-*.md`) define role identity only (~100 lines each)
- **Context files** (`.claude/adaptive-team-context/`) hold project-specific knowledge — populated by `/adaptive-team-setup`
- **Learned files** (`.claude/adaptive-team-learned/`) hold accumulated lessons from HITL self-healing
- **Rules** (`.claude/rules/adaptive-team-rules.md`) define team process

Agents read their role file, then load context and learned files at startup. This keeps agent definitions thin and reusable while project knowledge and lessons are isolated and replaceable.

## Key Principles

- **Adaptive learning** — every failure becomes a lesson in the right `adaptive-team-learned/` file
- **Root-cause routing** — PO diagnoses whether a problem is requirements, architecture, testing, process, or project-wide policy (CLAUDE.md)
- **One team per session** — add work to existing teams, don't rebuild
- **Task-scoped devs** — fresh agent per task, no context carry-over
- **Persistent reviewers** — architect and sdet maintain context across tasks
- **Worktree isolation** — all devs work in worktrees, merge only after review

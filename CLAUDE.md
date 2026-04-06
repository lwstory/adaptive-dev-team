# CLAUDE.md

## Project Overview

This repository contains a reusable Claude Code agent team model for structured software development. The team follows a Product Owner-led workflow with Architect and SDET Lead review gates.

## Development Work — Use the Dev Team Skill

**When technical implementation is requested**, use `/dev-team` to spin up a proper team. Do NOT implement code changes directly on the main thread.

The skill creates a Product Owner-led team with Architect, SDET Lead, and disposable Dev agents following the roles and review gates defined in `.claude/rules/team-rules.md`.

## Working with This Repository

- Agent role files in `.claude/agents/` define how each specialized agent should behave
- Rules in `.claude/rules/` define team processes and constraints
- Skills in `.claude/skills/` define pipeline commands
- All agents should read their role file before starting work
- The PO runs on the main thread; all other roles are subagents
- Devs always work in worktrees for isolation
- Every task passes through Architect + SDET Lead review before acceptance

## Key Principles

- **No silent workarounds** — surface operational failures and fix them in role files
- **HITL self-healing** — every repeated failure becomes a permanent rule improvement
- **One team per session** — add work to the existing team, don't rebuild
- **Task-scoped devs** — fresh agent per task, no context carry-over
- **Persistent reviewers** — Architect and SDET Lead stay alive across tasks

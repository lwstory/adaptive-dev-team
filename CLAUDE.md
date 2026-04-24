# CLAUDE.md

## Project Overview

This repository contains the Adaptive Dev Team — a self-healing Claude Code team for structured software development with human-in-the-loop learning.

## Skills (user-facing)

| User wants to... | Use |
|------------------|-----|
| Bootstrap the framework into a project | `/adaptive-team-init` |
| Discuss approach with the team (no code written) | `/adaptive-team-consult` |
| Build something | `/adaptive-team-implement` |
| Capture a lesson from something that went wrong | `/adaptive-team-learning-moment` |

## Team Composition

- **product-manager** (PM) — main thread; orchestrates; never reads or writes code
- **architect** — persistent reviewer; owns permission/authz lens and design coherence
- **sdet** — persistent reviewer; owns test strategy and coverage
- **llm-expert** — on-demand specialist for LLM / prompt / agent-design concerns
- **database** — on-demand specialist covering MySQL, Neo4j, and document stores
- **curious** — on-demand ideator; generates improvement proposals across roles and hands them to specialists for vetting
- **dev** — task-scoped, fresh per task, always in a worktree

Referred to as **team members** (or teammates), not sub-agents.

## Architecture

- **Agent files** (`.claude/agents/adaptive-team-*.md`) — role identity only, ~80-100 lines each
- **Context files** (`.claude/adaptive-team-context/`) — project-specific knowledge, populated by `/adaptive-team-init`
- **Learned files** (`.claude/adaptive-team-learned/`) — accumulated lessons from HITL self-healing
- **Rules** (`.claude/rules/adaptive-team-rules.md`) — team process, ≤200 lines
- **Session state** (`.claude/adaptive-team-state/current-session.md`) — PM's durable memory (survives compaction)
- **Reviews** (`.claude/adaptive-team-reviews/`) — reviewer findings per task; PM never reads these

## Key Principles

- **PM context discipline** — PM never reads/edits source, never receives review findings (verdicts only). Enforced by tool palette in the PM agent file.
- **Deterministic team names** — `architect`, `sdet`, `llm-expert`, `database`, `dev` — addressable by name across compaction.
- **Shift-left briefings** — relevant reviewers brief the dev in plain English before work starts. Briefings teach the *mentality*, not the code.
- **Reviewer self-reflection** — when dev work is UNSATISFIED, the *reviewer* diagnoses whether it briefed inadequately or the dev fell short. Dev never self-assesses.
- **User-approved learning** — every lesson routes through the user before being written to a learning file.
- **No CLAUDE.md auto-promotion** — additions only when a specific issue at that moment warrants project-wide policy.
- **Deterministic recovery** — PM rebuilds session state from rules + state file + pm-lessons + `TaskList` after compaction.

## PM Recovery Protocol

If you are the PM and sense drift (forgotten team members, unclear in-flight state, unsure of a rule): before taking any action, read in order:

1. `.claude/rules/adaptive-team-rules.md`
2. `.claude/adaptive-team-state/current-session.md`
3. `.claude/adaptive-team-learned/pm-lessons.md`

Then run `TaskList`. Only then decide the next step. **Never guess.**

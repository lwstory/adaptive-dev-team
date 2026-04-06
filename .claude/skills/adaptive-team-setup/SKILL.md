---
name: adaptive-team-setup
description: "Reconfigure the Adaptive Dev Team for this project. Re-runs the setup wizard to update context files."
argument-hint: "[optional: 'reset' to clear and regenerate, or specific area like 'testing']"
---

# Adaptive Team Setup

Re-runs the setup wizard portion of `/adaptive-team-init`. Use this to reconfigure after major stack changes, or to update a specific area.

**For first-time installation, use `/adaptive-team-init` instead — it handles both file installation and setup.**

## When to Use

- After major refactoring that changes project structure or tech stack
- To update a specific area (e.g., `/adaptive-team-setup testing`)
- With `reset` to clear all context files and regenerate from scratch

## Pipeline

Runs Steps 5-9 from `/adaptive-team-init`:

1. **Auto-discover** the project (language, structure, tests, build, infrastructure)
2. **Present defaults and ask** targeted questions
3. **Write context files** to `.claude/adaptive-team-context/`
4. **Validate** file sizes and content quality
5. **Confirm** with user

If called with `reset`, clears existing context files before regenerating. Never touches `adaptive-team-learned/`.

See `/adaptive-team-init` for the full pipeline details.

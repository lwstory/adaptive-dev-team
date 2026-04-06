# HITL Self-Healing Process

## Overview

Human-in-the-Loop (HITL) self-healing is the principle that **every operational failure becomes a permanent improvement** to the team's role files and rules. The team gets better over time — no silent workarounds, no repeated mistakes.

## How It Works

When something goes wrong in the team workflow:

1. **The PO stops and describes the problem** — what broke, what the impact is, what caused it
2. **The PO asks the user**: "Should we fix this now by updating role/rule files, or add a task to address after the current work?"
3. **If fix now**: update the relevant `.claude/` files to prevent recurrence, then resume
4. **If defer**: create a task with a clear description of the issue and the proposed fix

## What Triggers Self-Healing

| Trigger | Example Fix |
|---------|-------------|
| Agent goes idle without delivering work | Add "never finish silently" to role file communication rules |
| Team creation fails due to stale state | Add recovery procedure to team-rules.md |
| Build fails due to cached layers | Add build verification to dev checklist |
| Agent reviews the wrong resource | Add disambiguation section to role file |
| Merge ordering causes broken main branch | Add merge sequencing guidance to team-rules.md |
| Dev misses a test pattern consistently | Add the pattern to dev.md testing requirements |
| Architect misses an abstraction leak type | Add the check to architect.md review checklist |
| Any pattern that repeats twice | It MUST be codified in a rule/role file |

## The Two-Strike Rule

If a problem happens **once**, it's noted. If it happens **twice**, it gets codified:

- First occurrence: PO notes the issue and describes the fix
- Second occurrence: The fix MUST be written into the appropriate role file, rule file, or CLAUDE.md before continuing

## Where Fixes Go

| Problem Area | Fix Location |
|-------------|--------------|
| Agent communication | Role file `## Communication` section |
| Team lifecycle | `.claude/rules/team-rules.md` |
| Code quality gaps | `.claude/agents/dev.md` or `.claude/agents/architect.md` |
| Test quality gaps | `.claude/agents/sdet.md` or `.claude/agents/dev.md` |
| Build/deploy issues | `.claude/agents/dev.md` or `.claude/agents/architect.md` |
| Process gaps | `.claude/rules/team-rules.md` or role file `## Workflow` |
| Project-specific | `CLAUDE.md` |

## Key Principles

- **Never absorb failures silently** — the PO must surface every operational issue
- **The user decides timing** — fix now or defer, but the issue must be raised
- **Fixes are permanent** — written into files that persist across sessions
- **Role files evolve** — they are living documents, not static templates
- **Continuous improvement owns both sides** — each reviewer role (Architect, SDET Lead) can recommend improvements to their own file, to each other's files, and to the dev role file
- **Devs don't self-assess** — they flag gaps in `## Issues Found`; reviewers decide if a role file update is warranted

## Anti-Patterns

- Silently working around a problem without telling the user
- Noting a problem but not proposing a fix
- Proposing a fix verbally but not writing it into a file
- Writing a fix into CLAUDE.md when it belongs in a role file (or vice versa)
- Waiting until the end of a session to address accumulated issues

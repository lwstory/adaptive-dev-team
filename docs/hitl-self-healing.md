# HITL Self-Healing Process

## Overview

Human-in-the-Loop (HITL) self-healing is the principle that **every operational failure becomes a permanent improvement**. Lessons accumulate in `.claude/adaptive-team-learned/` — agent role files stay thin and stable. The team gets better over time with no silent workarounds.

## How It Works

1. **Something goes wrong** — bad output, agent failure, user dissatisfaction, process breakdown
2. **The PO diagnoses root cause** — was this a requirements miss, design gap, test gap, implementation issue, or process failure?
3. **The PO asks the user**: fix now or defer?
4. **If fix now**: write the lesson to the correct file in `.claude/adaptive-team-learned/`
5. **If defer**: create a task with the diagnosis and proposed fix

## Root-Cause Routing

The PO routes each lesson to the file that matches the root cause:

| Root Cause | Lesson File | Examples |
|-----------|-------------|----------|
| **Requirements/scope** | `po-lessons.md` | Misunderstood intent, stories too broad, wrong priority |
| **Design/architecture** | `architect-lessons.md` | Wrong abstraction, missed dependency, scalability issue |
| **Test coverage/quality** | `sdet-lessons.md` | Missed edge case, wrong test layer, insufficient coverage |
| **Implementation** | `dev-lessons.md` | Coding pattern issue, build problem, missed convention |
| **Team process** | `team-lessons.md` | Communication breakdown, review gate failure, coordination |

**The PO does NOT dump everything into `po-lessons.md`.** A bug that slipped through testing is an SDET lesson, not a PO lesson. A design that didn't scale is an architect lesson. Accurate diagnosis is critical — the wrong routing means the wrong agent learns.

## The Two-Strike Rule

- First occurrence: PO notes the issue and describes the fix
- Second occurrence: the fix MUST be written to `adaptive-team-learned/` before continuing

## What Triggers Self-Healing

| Trigger | Root Cause | Lesson File |
|---------|-----------|-------------|
| Agent goes idle without delivering work | Process | `team-lessons.md` |
| Team creation fails due to stale state | Process | `team-lessons.md` |
| Build fails due to cached layers | Implementation | `dev-lessons.md` |
| Dev misses a test pattern consistently | Testing | `sdet-lessons.md` |
| Architect misses an abstraction leak | Architecture | `architect-lessons.md` |
| User doesn't like the result | Requirements | `po-lessons.md` |
| Merge ordering causes broken main | Process | `team-lessons.md` |

## Key Principles

- **Never absorb failures silently** — the PO must surface every issue
- **Diagnose before routing** — accurate root cause determines where the lesson goes
- **The user decides timing** — fix now or defer, but the issue must be raised
- **Lessons persist across sessions** — `adaptive-team-learned/` files carry forward
- **Agent files stay stable** — lessons go to `adaptive-team-learned/`, never to agent role files
- **Devs don't self-assess** — they flag gaps in `## Issues Found`; reviewers decide the lesson

## Anti-Patterns

- Silently working around a problem without telling the user
- Noting a problem but not proposing a fix
- Proposing a fix verbally but not writing it to a file
- Routing every lesson to `po-lessons.md` instead of diagnosing root cause
- Modifying agent role files instead of writing to `adaptive-team-learned/`
- Waiting until the end of a session to address accumulated issues

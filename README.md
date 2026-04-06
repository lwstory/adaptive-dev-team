# Claude Dev Team

A reusable agent team model for Claude Code that implements a structured software development workflow with human-in-the-loop (HITL) self-healing.

## What This Is

A set of Claude Code agent definitions, rules, and skills that spin up a complete development team:

| Role | Purpose |
|------|---------|
| **Product Owner (PO)** | Orchestrates work — epics, stories, delegation, acceptance |
| **Architect** | Design guidance, code review, architectural quality gate |
| **SDET Lead** | Test strategy, test review, coverage verification |
| **Dev** | Implementation, testing, commits |

## Key Features

- **Worktree isolation** — every dev works in a git worktree, merges only after review
- **Review gates** — Architect + SDET Lead must both sign off before code is accepted
- **HITL self-healing** — operational failures become permanent improvements to role files
- **Task-scoped devs** — fresh context per task prevents bias and enables parallelism
- **Persistent reviewers** — Architect and SDET Lead maintain context across tasks

## Prerequisites

This framework uses [Claude Code Agent Teams](https://docs.anthropic.com/en/docs/claude-code/agent-teams), which enables multi-agent coordination with peer-to-peer messaging, shared task lists, and independent context windows per teammate.

### Requirements

| Requirement | Details |
|-------------|---------|
| **Claude Code** | v2.1.32 or later (`claude --version`) |
| **Agent Teams** | Experimental feature — must be enabled (see below) |
| **Git** | Required — worktrees are used for dev isolation |
| **Existing project** | The team needs a codebase to work on with an existing test suite |

### Enable Agent Teams

Agent Teams is experimental and disabled by default. Enable it by adding the environment variable to your Claude Code settings:

**Option A: Global settings** (`~/.claude/settings.json`):
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Option B: Per-project** (`.claude/settings.json` in your project root):
```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Option C: Shell environment**:
```bash
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
```

### How Teams Differ from Subagents

If you've used Claude Code's `Agent` tool (subagents), Teams are a higher-level primitive:

| | Subagents (`Agent` tool) | Agent Teams |
|---|---|---|
| **Context** | Shares parent's session | Independent context window per teammate |
| **Communication** | Reports back to parent only | Peer-to-peer messaging between teammates |
| **Task tracking** | No shared state | Shared task list across team |
| **Lifecycle** | Ephemeral | Persistent for the session (or task-scoped) |
| **Coordination** | Parent orchestrates | Lead + teammates self-coordinate |

This framework uses Teams because the Architect and SDET Lead need to message each other directly, maintain independent context, and persist across multiple task cycles.

### Known Limitations

- One team per session (cannot run two teams simultaneously)
- No nested teams (teammates cannot spawn sub-teams)
- Token costs scale linearly with teammates — expect higher usage than single-session workflows
- Persistent reviewers (Architect, SDET Lead) may hit context limits after 5-6 detailed review cycles in a single session

## Setup

### 1. Copy into your project

Copy the `.claude/` directory into your project root:

```bash
cp -r claude-dev-team/.claude/ your-project/.claude/
```

Also copy or merge the `CLAUDE.md` into your project root — it tells Claude Code to use `/dev-team` for implementation work.

### 2. Run `/project-context` to auto-configure

Once the files are in your project, open Claude Code in your project directory and run:

```
/project-context
```

This skill scans your codebase and outputs a structured context summary covering:
- Tech stack (languages, frameworks, versions)
- Project structure (directories, modules)
- Test infrastructure (frameworks, patterns, current coverage)
- Build system (Docker, CI/CD, build commands)
- Code patterns (DI, config, error handling conventions)

### 3. Paste the context into agent files

Take the output from `/project-context` and use it to fill in the **[CUSTOMIZE]** sections in each agent file (these are bold-formatted markers visible in both raw and rendered markdown):

| Agent File | What to customize |
|-----------|-------------------|
| `architect.md` | Tech stack, architecture docs, review checklist items specific to your stack |
| `sdet.md` | Test frameworks, pyramid targets, coverage thresholds, mock/fake patterns |
| `dev.md` | Code standards, test requirements, build/commit commands |
| `product-owner.md` | Project description, reference doc paths |

### 4. Use `/dev-team` to start building

```
/dev-team Add a new REST endpoint for user profiles
```

The skill spins up the full team — PO writes stories, Architect provides guidance, Dev builds in a worktree, reviewers sign off, then it merges.

## How It Works

```
User request
  → PO creates epics/stories/tasks
    → Architect provides design guidance
      → Dev implements in worktree
        → Architect reviews code → SATISFIED/UNSATISFIED
        → SDET Lead reviews tests → SATISFIED/UNSATISFIED
          → Both SATISFIED → PO accepts → Dev commits and merges
```

Max 2 review cycles per task. Escalates to user after 2 rejections.

## Self-Healing (HITL)

When something goes wrong — agents not responding, broken state, communication failures — the PO stops and engages the user:

1. Describes the problem clearly
2. Asks: fix now (update role files) or defer (create a task)?
3. Every operational failure becomes a permanent improvement

The team gets better over time. No silent workarounds.

## Structure

```
.claude/
  agents/
    product-owner.md   — PO role definition
    architect.md       — Architect role + review checklist
    sdet.md            — SDET Lead role + test review checklist
    dev.md             — Dev role + implementation standards
  rules/
    team-rules.md      — team composition, lifecycle, worktrees, review gates
  skills/
    dev-team/
      SKILL.md         — the skill that spins up the team
    project-context/
      SKILL.md         — auto-discover project context for agent customization
docs/
  hitl-self-healing.md — detailed HITL self-healing process documentation
```

## License

MIT

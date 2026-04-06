---
name: adaptive-team-init
description: "Initialize the Adaptive Dev Team in your current project. Pulls files from the repo, discovers your codebase, and configures agents."
argument-hint: "[optional: git ref/tag to pull, defaults to master]"
---

# Adaptive Team Init

Initializes the Adaptive Dev Team framework in your current project. Clones the repo, copies files, discovers your codebase, and configures the agents — all in one flow.

**This skill is installed globally** (`~/.claude/skills/adaptive-team-init/`) so it's available in any project.

## Prerequisites

- Git installed
- Claude Code with Agent Teams enabled (see repo README)

## Pipeline

### Step 1: Check for Existing Installation

Check if `.claude/agents/adaptive-team-product-owner.md` already exists.

- **If found**: Ask the user — "Adaptive Dev Team is already installed. Overwrite with latest? (This won't touch your adaptive-team-context/ or adaptive-team-learned/ files.)" If user declines, skip to Step 5 (setup wizard) to reconfigure.
- **If not found**: Proceed.

### Step 2: Clone to Temp

```bash
TEMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/lwstory/adaptive-dev-team.git "$TEMP_DIR/adaptive-dev-team"
```

If a git ref/tag was provided as an argument, check out that ref.

### Step 3: Copy Files

```bash
mkdir -p .claude/agents .claude/rules .claude/adaptive-team-context .claude/adaptive-team-learned

# Agent files
cp "$TEMP_DIR/adaptive-dev-team/.claude/agents/adaptive-team-"*.md .claude/agents/

# Rules
cp "$TEMP_DIR/adaptive-dev-team/.claude/rules/adaptive-team-rules.md" .claude/rules/

# Skills (start, plan, implement, setup)
for skill in adaptive-team-start adaptive-team-plan adaptive-team-implement adaptive-team-setup; do
  mkdir -p ".claude/skills/$skill"
  cp "$TEMP_DIR/adaptive-dev-team/.claude/skills/$skill/SKILL.md" ".claude/skills/$skill/"
done

# Docs
mkdir -p docs
cp "$TEMP_DIR/adaptive-dev-team/docs/hitl-self-healing.md" docs/

# Empty dirs if no content yet
[ -z "$(ls -A .claude/adaptive-team-context 2>/dev/null)" ] && touch .claude/adaptive-team-context/.gitkeep
[ -z "$(ls -A .claude/adaptive-team-learned 2>/dev/null)" ] && touch .claude/adaptive-team-learned/.gitkeep
```

**Never overwrite** `adaptive-team-context/` or `adaptive-team-learned/` contents.

### Step 4: Clean Up Temp

```bash
rm -rf "$TEMP_DIR"
```

### Step 5: Auto-Discover Project

Scan the project to detect the tech stack and pre-populate defaults:

- **Language/framework**: read package.json, *.csproj, pyproject.toml, Cargo.toml, go.mod, etc.
- **Project structure**: list directories (2 levels), identify source vs test vs docs
- **Test setup**: find test dirs, config, frameworks, mock/fake patterns
- **Build system**: Dockerfiles, CI/CD config, build scripts
- **Infrastructure**: database migrations, message queues, caches

Auto-detect defaults:
- `package.json` → build: `npm test`, framework: Jest/Vitest
- `*.csproj` → build: `dotnet test`, framework: xUnit/NUnit
- `pyproject.toml` → build: `pytest`
- `Cargo.toml` → build: `cargo test`
- `go.mod` → build: `go test ./...`

### Step 6: Ask Targeted Questions

Present auto-detected defaults. Only ask what can't be inferred:

1. **Architecture style** — Clean Architecture, MVC, microservices, monolith, serverless?
2. **Code conventions** — DI patterns, error handling strategy, config management?
3. **Test naming convention** — (pre-filled from detected test files if possible)
4. **Coverage targets** — 80% overall? 90% core? No formal targets?
5. **Build/verify command** — (pre-filled, confirm or override)
6. **Documentation locations** — (pre-filled if docs/ detected)
7. **Common quality issues** — what problems come up most?
8. **Merge strategy** — PRs or direct merge to main branch?

### Step 7: Write Context Files

Write to `.claude/adaptive-team-context/`:

| File | Content |
|------|---------|
| `tech-stack.md` | Languages, frameworks, versions, infrastructure |
| `architecture.md` | Patterns, conventions, dependency rules |
| `testing.md` | Pyramid targets, naming, coverage, mock patterns, build command |
| `review-checklist.md` | Stack-specific review items for the architect |

### Step 8: Validate

- Each context file under 100 lines
- No duplication across files
- Context is specific enough to be useful

### Step 9: Confirm

```
Adaptive Dev Team initialized and configured.

Files installed:
  - 4 agent files, 1 rules file, 4 skills
  - docs/hitl-self-healing.md

Project context written:
  - tech-stack.md: [summary]
  - architecture.md: [summary]
  - testing.md: [summary]
  - review-checklist.md: [summary]

Ready to go. Use:
  /adaptive-team-start   — start a session with the full team
  /adaptive-team-plan    — design review before building
  /adaptive-team-implement — build a feature
```

## What Gets Copied

| Copied | Not Copied |
|--------|-----------|
| Agent files (`adaptive-team-*.md`) | README.md, CLAUDE.md, LICENSE |
| Rules (`adaptive-team-rules.md`) | .gitignore |
| Skills (start, setup, plan, implement) | adaptive-team-context/ contents (generated fresh) |
| docs/hitl-self-healing.md | adaptive-team-learned/ contents (accumulated lessons) |

## Global Installation

```
~/.claude/skills/adaptive-team-init/SKILL.md
```

Install:
```bash
mkdir -p ~/.claude/skills/adaptive-team-init
curl -o ~/.claude/skills/adaptive-team-init/SKILL.md \
  https://raw.githubusercontent.com/lwstory/adaptive-dev-team/master/.claude/skills/adaptive-team-init/SKILL.md
```

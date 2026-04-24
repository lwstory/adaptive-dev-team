---
name: adaptive-team-init
description: "Initialize the Adaptive Dev Team in your current project. Pulls files from the repo, discovers your codebase, and configures the team."
argument-hint: "[optional: git ref/tag to pull, defaults to master]"
---

# Adaptive Team Init

Initializes the Adaptive Dev Team framework in your current project. Clones the repo, copies files, discovers your codebase, and configures the team — all in one flow.

**Installed globally** (`~/.claude/skills/adaptive-team-init/`) so it's available in any project.

## Pipeline

### Step 1: Check Dependencies

For each check, report the result and help fix if missing.

**Required:**

```bash
# Git installed
git --version

# Claude Code version (needs Agent Teams support)
claude --version
# If < v2.1.32: "Claude Code v2.1.32+ required. Run: claude update"

# Agent Teams enabled
cat ~/.claude/settings.json 2>/dev/null | grep -q "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS" \
  || cat .claude/settings.json 2>/dev/null | grep -q "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS"
```

If Agent Teams is not enabled, ask the user to enable it. Add to the appropriate `settings.json`:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

**Optional (report):**

```bash
# Git repo
git rev-parse --is-inside-work-tree
# If not: "This directory is not a git repo. The team uses worktrees — initialize with git init?"

# Source files present
ls -A | grep -v "^\.git$" | head -1
# If empty: note the setup wizard will skip auto-detect; can re-run later once code exists.
```

Report a dependency summary before proceeding.

### Step 2: Check for Existing Installation

Look for `.claude/agents/adaptive-team-product-manager.md`.

- **If found**: Ask — "Adaptive Dev Team is already installed. Overwrite with latest? (This won't touch your `adaptive-team-context/` or `adaptive-team-learned/` files.)"
- **If declined**: skip to the setup wizard (Step 6) to reconfigure.

### Step 3: Clone to Temp

```bash
TEMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/lwstory/adaptive-dev-team.git "$TEMP_DIR/adaptive-dev-team"
```

If a git ref/tag was provided, check out that ref.

### Step 4: Copy Files

```bash
mkdir -p .claude/agents .claude/rules .claude/adaptive-team-context \
         .claude/adaptive-team-learned .claude/adaptive-team-state \
         .claude/adaptive-team-reviews

# Agent files (all adaptive-team-*.md)
cp "$TEMP_DIR/adaptive-dev-team/.claude/agents/adaptive-team-"*.md .claude/agents/

# Rules
cp "$TEMP_DIR/adaptive-dev-team/.claude/rules/adaptive-team-rules.md" .claude/rules/

# Skills (consult, implement, learning-moment — init is already installed globally)
for skill in adaptive-team-consult adaptive-team-implement adaptive-team-learning-moment; do
  mkdir -p ".claude/skills/$skill"
  cp "$TEMP_DIR/adaptive-dev-team/.claude/skills/$skill/SKILL.md" ".claude/skills/$skill/"
done

# Seed runtime directories from templates (only if missing)
# Note: runtime dirs (adaptive-team-learned/, adaptive-team-state/, adaptive-team-reviews/)
# are NOT committed to the framework repo. They hold per-project accumulated state. Templates
# in .claude/templates/ are the canonical starting content; this step materializes them locally.
mkdir -p .claude/adaptive-team-learned .claude/adaptive-team-state .claude/adaptive-team-reviews

# Lesson seed files (only if missing — never overwrite accumulated lessons)
for lesson in pm-lessons architect-lessons sdet-lessons llm-lessons database-lessons dev-lessons team-lessons curious-lessons; do
  [ ! -f ".claude/adaptive-team-learned/$lesson.md" ] && \
    cp "$TEMP_DIR/adaptive-dev-team/.claude/templates/$lesson.md.template" ".claude/adaptive-team-learned/$lesson.md"
done

# Session state + pending PM lessons
[ ! -f ".claude/adaptive-team-state/current-session.md" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/.claude/templates/current-session.md.template" .claude/adaptive-team-state/current-session.md
[ ! -f ".claude/adaptive-team-state/pending-pm-lessons.md" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/.claude/templates/pending-pm-lessons.md.template" .claude/adaptive-team-state/pending-pm-lessons.md

# Reviews README
[ ! -f ".claude/adaptive-team-reviews/README.md" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/.claude/templates/reviews-README.md.template" .claude/adaptive-team-reviews/README.md

# Project permissions (only if missing — never overwrite user's settings)
[ ! -f ".claude/settings.json" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/.claude/settings.json" .claude/

# User-global PM-user-interaction lessons (only if missing — lives at ~/.claude/, not in project)
[ ! -f "$HOME/.claude/pm-user-lessons.md" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/.claude/templates/pm-user-lessons.md.template" "$HOME/.claude/pm-user-lessons.md"

# Docs
mkdir -p docs
[ -f "$TEMP_DIR/adaptive-dev-team/docs/hitl-self-healing.md" ] && \
  cp "$TEMP_DIR/adaptive-dev-team/docs/hitl-self-healing.md" docs/

# Empty dir markers
[ -z "$(ls -A .claude/adaptive-team-context 2>/dev/null)" ] && touch .claude/adaptive-team-context/.gitkeep
```

**Never overwrite** `adaptive-team-context/` or `adaptive-team-learned/` contents.

### Step 5: Clean Up Temp

```bash
rm -rf "$TEMP_DIR"
```

### Step 6: Auto-Discover Project

Scan the project to detect tech stack and pre-populate defaults:

- **Language/framework**: read `package.json`, `*.csproj`, `pyproject.toml`, `Cargo.toml`, `go.mod`, etc.
- **Project structure**: list directories (2 levels), identify source vs test vs docs
- **Test setup**: find test dirs, config, frameworks, mock patterns
- **Build system**: Dockerfiles, CI/CD, build scripts
- **Infrastructure**: migrations, message queues, caches

Auto-detect defaults:
- `package.json` → `npm test`, Jest/Vitest
- `*.csproj` → `dotnet test`, xUnit/NUnit
- `pyproject.toml` → `pytest`
- `Cargo.toml` → `cargo test`
- `go.mod` → `go test ./...`

### Step 7: Targeted Questions

Present auto-detected defaults. Only ask what can't be inferred:

1. Architecture style — Clean Architecture, MVC, microservices, monolith, serverless?
2. Code conventions — DI patterns, error handling, config management?
3. Test naming convention — (pre-fill from detected tests if possible)
4. Coverage targets — 80% overall? 90% core? No formal targets?
5. Build/verify command — (pre-fill, confirm or override)
6. Documentation locations — (pre-fill if `docs/` detected)
7. Common quality issues — what problems recur?
8. Merge strategy — PRs or direct merge?

### Step 8: Write Context Files

| File | Content |
|------|---------|
| `tech-stack.md` | Languages, frameworks, versions, infrastructure |
| `architecture.md` | Patterns, conventions, dependency rules |
| `testing.md` | Pyramid targets, naming, coverage, mock patterns, build command |
| `review-checklist.md` | Stack-specific review items for the architect |

### Step 9: Validate

- Each context file under 100 lines
- No duplication across files
- Content specific enough to be useful

### Step 10: Confirm

```
Adaptive Dev Team initialized and configured.

Files installed:
  - 6 agent files (PM, architect, sdet, dev, llm-expert, database)
  - 1 rules file
  - 3 skills (consult, implement, learning-moment)
  - 7 learning file seeds
  - session state template, reviews README
  - project settings.json (if not already present)

Project context written:
  - tech-stack.md: [summary]
  - architecture.md: [summary]
  - testing.md: [summary]
  - review-checklist.md: [summary]

Ready to go. Use:
  /adaptive-team-consult <topic>    — design/approach discussion, no code
  /adaptive-team-implement <task>   — full implementation pipeline
  /adaptive-team-learning-moment    — capture a lesson from an issue
```

## What Gets Copied

| Copied | Not Copied |
|--------|-----------|
| Agent files (`adaptive-team-*.md`) | README.md, CLAUDE.md, LICENSE |
| Rules (`adaptive-team-rules.md`) | `.gitignore` |
| Skills (consult, implement, learning-moment) | `adaptive-team-context/` contents (generated fresh) |
| Learning file seeds (only if missing) | Project-specific learned content (preserved) |
| Session state template, reviews README (only if missing) | |
| Project `settings.json` (only if missing) | |

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

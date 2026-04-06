---
name: adaptive-team-init
description: "Initialize the Adaptive Dev Team in your current project. Pulls the latest files from the repo and runs setup."
argument-hint: "[optional: git ref/tag to pull, defaults to main]"
---

# Adaptive Team Init

Initializes the Adaptive Dev Team framework in your current project directory. Clones the repo to a temp directory, copies the relevant files into your project's `.claude/` folder, cleans up, and prompts you to run `/adaptive-team-setup`.

**This skill is installed globally** (`~/.claude/skills/adaptive-team-init/`) so it's available in any project.

## Prerequisites

- Git installed
- Claude Code with Agent Teams enabled (see repo README for setup)

## Pipeline

### Step 1: Check for Existing Installation

Check if `.claude/agents/adaptive-team-product-owner.md` already exists in the current project.

- **If found**: Ask the user — "Adaptive Dev Team is already installed. Overwrite with latest? (This won't touch your adaptive-team-context/ or adaptive-team-learned/ files.)"
- **If not found**: Proceed.

### Step 2: Clone to Temp

```bash
TEMP_DIR=$(mktemp -d)
git clone --depth 1 https://github.com/lwstory/adaptive-dev-team.git "$TEMP_DIR/adaptive-dev-team"
```

If a git ref/tag was provided as an argument, check out that ref:
```bash
cd "$TEMP_DIR/adaptive-dev-team" && git checkout <ref>
```

### Step 3: Copy Files

Create directories and copy files into the current project:

```bash
# Create directories
mkdir -p .claude/agents .claude/rules .claude/adaptive-team-context .claude/adaptive-team-learned

# Copy agent files
cp "$TEMP_DIR/adaptive-dev-team/.claude/agents/adaptive-team-"*.md .claude/agents/

# Copy rules
cp "$TEMP_DIR/adaptive-dev-team/.claude/rules/adaptive-team-rules.md" .claude/rules/

# Copy skills (setup, plan, implement)
for skill in adaptive-team-setup adaptive-team-plan adaptive-team-implement; do
  mkdir -p ".claude/skills/$skill"
  cp "$TEMP_DIR/adaptive-dev-team/.claude/skills/$skill/SKILL.md" ".claude/skills/$skill/"
done

# Copy docs
mkdir -p docs
cp "$TEMP_DIR/adaptive-dev-team/docs/hitl-self-healing.md" docs/

# Add .gitkeep to empty dirs (if they don't have content yet)
[ -z "$(ls -A .claude/adaptive-team-context 2>/dev/null)" ] && touch .claude/adaptive-team-context/.gitkeep
[ -z "$(ls -A .claude/adaptive-team-learned 2>/dev/null)" ] && touch .claude/adaptive-team-learned/.gitkeep
```

**Never overwrite** `adaptive-team-context/` or `adaptive-team-learned/` contents — those hold project-specific knowledge and accumulated lessons.

### Step 4: Clean Up

```bash
rm -rf "$TEMP_DIR"
```

### Step 5: Prompt Setup

Tell the user:
```
Adaptive Dev Team installed. Files added to .claude/:
- 4 agent files (agents/adaptive-team-*.md)
- 1 rules file (rules/adaptive-team-rules.md)
- 3 skills (adaptive-team-setup, adaptive-team-plan, adaptive-team-implement)
- docs/hitl-self-healing.md

Next step: run /adaptive-team-setup to configure the agents for this project.
```

## What Gets Copied (and What Doesn't)

| Copied | Not Copied |
|--------|-----------|
| Agent files (`adaptive-team-*.md`) | README.md (repo docs, not project docs) |
| Rules (`adaptive-team-rules.md`) | CLAUDE.md (user's project has its own) |
| Skills (setup, plan, implement) | .gitignore (user's project has its own) |
| docs/hitl-self-healing.md | adaptive-team-context/ contents (project-specific) |
| Empty adaptive-team-context/ dir | adaptive-team-learned/ contents (accumulated lessons) |
| Empty adaptive-team-learned/ dir | |

## Global Installation

This skill file should be installed at:
```
~/.claude/skills/adaptive-team-init/SKILL.md
```

To install manually:
```bash
mkdir -p ~/.claude/skills/adaptive-team-init
cp .claude/skills/adaptive-team-init/SKILL.md ~/.claude/skills/adaptive-team-init/
```

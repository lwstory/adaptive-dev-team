---
name: project-context
description: "Gather project context to help team agents understand the codebase. Run this before or during team setup to populate agent System Context sections."
argument-hint: "[optional: specific area to focus on, e.g. 'testing', 'architecture', 'dependencies']"
---

# Project Context

Gathers project context so the dev team agents can be more specific to the actual codebase. This skill reads the project structure, tech stack, patterns, and conventions, then outputs a structured context summary that can be pasted into the `## System Context` sections of the agent files.

**Use this skill when setting up the dev team for a new project, or when agents need more codebase-specific context.**

## What It Discovers

| Area | How | Output |
|------|-----|--------|
| **Tech Stack** | Read package files (package.json, *.csproj, requirements.txt, Cargo.toml, go.mod, etc.) | Languages, frameworks, versions |
| **Project Structure** | List directories, find source roots | Directory tree, module layout |
| **Test Infrastructure** | Find test directories, test config, test frameworks | Test runner, patterns, pyramid shape |
| **Build System** | Find build scripts, Dockerfiles, CI config | Build commands, containerization |
| **Key Patterns** | Sample source files for DI, config, error handling patterns | Code conventions |
| **Documentation** | Find existing docs (README, architecture docs, ADRs) | Doc locations and summaries |
| **Dependencies** | Read lockfiles and dependency manifests | External service dependencies |

## Pipeline

### Step 1: Discover Project Type

Read the project root for manifest files to determine the tech stack:
- `package.json` → Node.js/TypeScript
- `*.csproj` / `*.sln` → .NET/C#
- `requirements.txt` / `pyproject.toml` → Python
- `Cargo.toml` → Rust
- `go.mod` → Go
- `pom.xml` / `build.gradle` → Java/Kotlin

### Step 2: Map Project Structure

```bash
# Get directory tree (2 levels deep)
find . -maxdepth 2 -type d -not -path '*/\.*' -not -path '*/node_modules/*' -not -path '*/bin/*' -not -path '*/obj/*'
```

### Step 3: Identify Test Setup

- Find test directories and test files
- Read test configuration (jest.config, xunit, pytest.ini, etc.)
- Count tests by type/category if categorized
- Identify mock/fake patterns

### Step 4: Check Build & Deploy

- Find Dockerfiles, docker-compose files
- Find CI/CD config (.github/workflows, .gitlab-ci.yml, Jenkinsfile)
- Find build scripts (Makefile, package.json scripts, etc.)

### Step 5: Sample Key Patterns

Read 2-3 representative source files to identify:
- Dependency injection patterns
- Configuration patterns
- Error handling patterns
- Logging patterns
- API/endpoint patterns

### Step 6: Output Context Summary

Output a structured summary that can be used to customize the `<!-- CUSTOMIZE -->` sections in the agent files:

```markdown
## Project Context Summary

### Tech Stack
- [language] [version] — [framework] [version]
- [database/cache/queue if applicable]
- [deployment target]

### Project Structure
[directory tree]

### Test Infrastructure
- Framework: [test framework]
- Patterns: [mock/fake patterns found]
- Current distribution: [test counts by type if available]

### Build System
- Build command: [command]
- Container: [yes/no, Dockerfile location]
- CI: [CI system, config location]

### Code Patterns
- DI: [pattern found]
- Config: [pattern found]
- Error handling: [pattern found]

### Key Documentation
- [doc path] — [what it covers]
```

## When to Use This Skill

- First time setting up the dev team for a project
- After major refactoring that changes project structure
- When agents are giving generic advice that should be project-specific
- When onboarding to an unfamiliar codebase

---
name: adaptive-team-setup
description: "Interactive setup wizard that discovers your project and configures the adaptive team agents for your specific codebase."
argument-hint: "[optional: 'reset' to reconfigure]"
---

# Adaptive Team Setup

Interactive wizard that discovers your codebase and configures the adaptive team agents by populating `.claude/adaptive-team-context/`. Agent files are never modified — all project-specific knowledge lives in the context folder.

**Run once when you first add the adaptive team to your project.**

## Pipeline

### Step 1: Auto-Discover and Set Defaults

Scan the project to detect the tech stack and pre-populate sensible defaults:

- **Language/framework**: read package.json, *.csproj, pyproject.toml, Cargo.toml, go.mod, etc.
- **Project structure**: list directories (2 levels), identify source vs test vs docs
- **Test setup**: find test dirs, config, frameworks, existing mock/fake patterns
- **Build system**: Dockerfiles, CI/CD config, build scripts
- **Infrastructure**: database migrations, message queues, caches

**Auto-detect defaults** (user reviews and overrides as needed):
- `package.json` detected → default build: `npm test`, test framework: Jest/Vitest (from devDependencies)
- `*.csproj` detected → default build: `dotnet test`, test framework: xUnit/NUnit (from PackageReference)
- `pyproject.toml` detected → default build: `pytest`, test framework: pytest
- `Cargo.toml` detected → default build: `cargo test`
- `go.mod` detected → default build: `go test ./...`

### Step 2: Present Defaults, Ask What Can't Be Inferred

Show auto-detected defaults and ask the user to confirm or override. Only ask questions where auto-detection can't provide an answer:

1. **Architecture style** — Clean Architecture, MVC, microservices, monolith, serverless?
2. **Code conventions** — DI patterns, error handling strategy, config management?
3. **Test naming convention** — (pre-filled from detected test files if possible)
4. **Coverage targets** — 80% overall? 90% core? No formal targets?
5. **Build/verify command** — (pre-filled from auto-detect, confirm or override)
6. **Documentation locations** — (pre-filled if docs/ or wiki/ detected)
7. **Common quality issues** — what problems come up most in this codebase?
8. **Merge strategy** — should devs create pull requests or merge directly to the main branch?

### Step 3: Write Context Files

Write the answers directly to `.claude/adaptive-team-context/`:

| File | Content | Sources |
|------|---------|---------|
| `tech-stack.md` | Languages, frameworks, versions, infrastructure | Auto-discover |
| `architecture.md` | Patterns, conventions, dependency rules, file placement | Auto-discover + Q1, Q2 |
| `testing.md` | Pyramid targets, naming, coverage, mock patterns, build command | Auto-discover + Q3, Q4, Q5 |
| `review-checklist.md` | Stack-specific review items for adaptive-team-architect | Q2, Q7 |
| `project-docs.md` | Where docs live, what they cover | Auto-discover + Q6 |

### Step 4: Validate

Run a best-practices check on the generated context files:
- Each file under 100 lines
- No duplication across files
- All agent cross-references valid
- Context is specific enough to be useful, not so generic it adds no value

### Step 5: Confirm

Show the user what was generated and ask if anything needs adjustment.

## When to Use

- **First setup**: once after copying `.claude/` into your project
- **After major changes**: run with `reset` if stack or architecture changes
- **Partial update**: specify an area (e.g., `/adaptive-team-setup testing`)

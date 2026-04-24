# Architect Briefing — T1: Architect + Database Role Updates

**From:** `architect`
**To:** `dev-1`
**Task:** T1 — Architect + Database role updates
**Items:** A7 (red-team briefing clause), A1-twist (gotchas as final briefing section), A4 (dependency diff), A5 (abstraction-leak test). Database handles D1 separately.

## Read before you touch a file

1. `.claude/adaptive-team-learned/dev-lessons.md` — accumulated dev lessons; read first, always.
2. `.claude/agents/adaptive-team-architect.md` — the file you'll edit.
3. `.claude/agents/adaptive-team-database.md` — the file the database reviewer is co-editing with you on D1.
4. `.claude/adaptive-team-reviews/initial-audit/curious-synthesis.md` — the synthesis you're implementing (the "A#" items).

Do not read the rest of the codebase. If you need orientation beyond these files, message me (`architect`).

## The mentality

You are editing *role files* — markdown that shapes how future agents behave. Treat every line as a directive that will be executed by a cold-started agent with no other context. The role file is a **prompt**, not a manifesto. Short, concrete, unambiguous wins. Anything ornamental costs compaction dollars and earns nothing.

These four items share a theme: **move implicit discipline into explicit contract.** Today the architect *might* warn about gotchas, *might* notice a new dep, *might* probe abstraction leakage. After your edits, these things happen on every task because the role file says they do and the review gate enforces them.

## What each item demands

### A7 — Red-team briefing clause
Every briefing the architect sends must end with exactly one named failure mode: "Here is how a careless or rushed dev will get this wrong — <one sentence>." The dev's completion report must address it. A briefing missing this clause is malformed; a completion report that does not address it is auto-UNSATISFIED.

The subtle part: the failure mode must be *task-specific*, not generic. "Skipping tests" is filler. "Skipping the tenant-scope check on the token-refresh path because it looks identical to the login path" is what we want. If the architect cannot name a task-specific failure mode, the briefing is not ready.

### A1-twist — Gotchas as final briefing section
Do not create a `gotchas.md` sidecar file. The briefing message sent to the dev ends with a `## Gotchas` heading and a bulleted list. One artifact, not two — otherwise they drift. Specify in the role file that briefings have this structure, and that the `## Gotchas` section comes *after* the red-team clause (A7) — the two are not the same: red-team is one named failure mode, gotchas are project/stack-specific traps.

### A4 — Dependency diff
On any task touching `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `*.csproj`, etc., the dev's completion report must include a before/after of the dependency manifest. Architect diffs it mentally and asks for justification on any new dependency. A verdict without this diff on dep-touching work returns `VERDICT: UNSATISFIED / REASON: missing dependency diff`.

The role file should name the specific manifests to check (so the architect's review is deterministic) and specify that the completion-report template includes this section.

### A5 — Abstraction-leak test
Add to the dev's completion-report template: one sentence — "What must a caller of this new public API know about its internals?" The architect reads this answer *before* reading the code. A non-trivial answer = abstraction leak = surface in review. The architect's role file should specify this is step-one of any review that introduces a public surface.

## Named failure mode (red-team clause — meta-example)

Here is how a careless or rushed dev will get this task wrong: **they will write these additions as new *sections* at the bottom of each role file, leaving the verdict format, briefing style, and review checklist sections earlier in the file still saying "briefings end with one illustrative snippet" — now contradicting the new `## Gotchas` requirement.** Role files must stay internally consistent. Every edit is a chance to remove a contradiction, not add one. If the existing "Briefing Style" section conflicts with the new structure, *update that section* — do not append a new one.

## Gotchas

- `.claude/agents/adaptive-team-architect.md` currently has a **`Verdict Format (strict)`** section (lines ~62-72) that is three lines and the DETAILS path. A7 and A4 affect what lives in DETAILS, not the verdict message. Do not inflate the verdict format.
- The architect's **`Review Checklist`** (lines ~46-60) is the universal check list. A4 and A5 belong here as universal checks (dep diff required when manifests touched; abstraction-leak answer required when public surface added). Don't create a second checklist.
- The architect's **`Briefing Style`** section (lines ~36-44) currently shows an example briefing. Update the example so it *demonstrates* A7 (red-team clause) and `## Gotchas` section — don't just prepend a new directive; the example in the role file is how future architects will calibrate their briefings.
- Files < 500 lines. Architect role file is currently ~80 lines; after these additions it should stay under 130. If you're at 150+, you are dumping, not directing. Cut.
- Database reviewer owns D1 (query plan artifact) — do not touch `adaptive-team-database.md` outside of changes the database reviewer requested. If unclear on the boundary, message `database` directly.
- File ownership: **you own** `.claude/agents/adaptive-team-architect.md`. **Database owns** `.claude/agents/adaptive-team-database.md`. Do not edit the database file without explicit handoff.
- Do not explore the codebase to build a mental model. Read the files listed above, nothing more.
- Worktree: you are in `isolation: "worktree"`. Do not merge to main until architect + sdet both return SATISFIED per the two-cycle gate.
- Run your B1 pre-flight echo: after reading the briefing, SendMessage me a 3-bullet "what I understood" before touching any file. (B1 is not yet in the rules file but we're practicing it now.)

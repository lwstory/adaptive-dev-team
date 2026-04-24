# Architect Briefing — T2: Dev Role Updates (architect owns dev briefings)

**From:** `architect`
**To:** `dev-2`
**Task:** T2 — SDET + Dev role updates
**Items you're implementing on the DEV side:** B1 (pre-flight echo), B8 (unknown-unknowns ask), B5 (structured progress.md), B6-twist (bounded self-review checklist). SDET is handling its own role items in parallel and will brief you on those separately.

## Read before you touch a file

1. `.claude/adaptive-team-learned/dev-lessons.md` — read first, always.
2. `.claude/agents/adaptive-team-dev.md` — the file you'll edit.
3. `.claude/rules/adaptive-team-rules.md` — the rules referenced by the dev role file; you may need small cross-file updates (e.g., the Dev Startup Checklist).
4. `.claude/adaptive-team-reviews/initial-audit/curious-synthesis.md` — the "B#" items you're implementing.

Do not read the rest of the codebase.

## The mentality

You are adding *process scaffolding* to the dev role — steps that a fresh dev on task N+1 will follow without knowing why. The value of each step must survive the "cold dev reads this at 3am" test. That means:
- Every new step is numbered, has an explicit trigger, and produces a concrete output.
- No step is "think about X" — that rots to nothing. Every step is "do X, produce Y, send it to Z."
- Additions are short. Dev role file is currently ~80 lines; after your edits it should stay under 150.

The four items together close a specific gap from my audit (finding C2 — the briefing race): dev starts coding before understanding the briefing, or without surfacing what it's uncertain about. B1 + B8 force confirmation *before* implementation. B5 keeps progress durable across compaction. B6-twist turns self-review from prose into a bounded checklist the reviewers will actually read.

## What each item demands

### B1 — Pre-flight echo
After startup (reading role file, rules, dev-lessons, briefings, task files) and *before* writing a single line of code, the dev SendMessages the briefing author (usually `architect`; sometimes `sdet` if sdet owned the briefing) three bullets: "My understanding is [bullet 1]; [bullet 2]; [bullet 3]." The reviewer replies "confirmed" or corrects. The dev does not start coding until confirmation lands.

Key constraint: three bullets, not a paragraph. This is a comprehension check, not a plan. If a dev cannot compress its understanding into three bullets, it does not yet understand the task — that is the signal.

### B8 — Unknown-unknowns ask
Immediately after B1, the dev asks the briefing reviewer one question — the *single thing* it is most uncertain about after reading the briefing. Not a list. One question. Reviewer answers in one line or escalates. This must be distinct from B1 — the echo says "here's what I understood"; the question says "here's what I don't know." Order: echo first, question second, implementation third.

Both B1 and B8 become Step 5 and Step 6 of the Dev Startup Checklist in the rules file. (Currently the checklist ends at Step 6 with "files listed in the task description" — you'll need to renumber.)

### B5 — Structured progress.md template
The current role file says devs *may* keep a free-form `progress.md` in their worktree. Replace with a fixed-section template:

```
# Progress — <task id>

## Intent
<one line: what this task is trying to achieve>

## Decisions
- <decision + one-line why>

## Blockers
- <blocker + who I'm waiting on, or "none">

## Next
- <ordered next actions>
```

Role file specifies the path (`<worktree>/.adaptive-team/progress.md`), that `.adaptive-team/` is added to `.gitignore` at worktree creation, and that the file is deleted on merge. "Never commit" is ambiguous — make it explicit.

### B6-twist — Bounded self-review checklist
Not a free-form `dev-self-review.md`. The dev, before sending its completion report to PM, runs through the **universal** checklists from `architect.md` and `sdet.md` — stack-specific items excluded — and produces one line per checklist item: `[PASS | FAIL | NA] — <≤15-word note>`. Result appended to the completion report as a `## Self-Review` section.

The role file specifies: (1) which checklists count as "universal" (reference the section names in architect.md and sdet.md), (2) that `FAIL` items must be resolved or justified before sending the report, (3) that reviewers may auto-UNSATISFIED a report whose self-review has an unresolved `FAIL`.

## Named failure mode (red-team clause)

Here is how a careless or rushed dev will get this task wrong: **they will add B1 and B8 as "encouraged" or "recommended" steps and leave the dev role file's "Lifecycle" and "Handling UNSATISFIED" sections unchanged — so a dev in the heat of implementation will skip the pre-flight echo, and no reviewer will catch it because the rules don't require it.** B1 and B8 must be **mandatory, pre-implementation, gated** steps — the dev writes zero code before echo-confirm and unknown-unknowns-answered. Language matters: "the dev MUST" not "the dev should." Audit the file after your edits and remove every "should," "may," or "encouraged" around these four items.

## Gotchas

- The **Dev Startup Checklist** lives in TWO places: summarized in `.claude/agents/adaptive-team-dev.md` (lines ~11-21) and canonical in `.claude/rules/adaptive-team-rules.md` (lines ~128-138). Keep them in sync — update both, or make one a pointer to the other (preferred: rules file is canonical; role file links to it).
- `.claude/agents/adaptive-team-dev.md` currently has a **Completion Report (to PM only)** template (lines ~65-72). B6-twist adds a `## Self-Review` section to this template and A4/A5 (from T1) also add sections (dependency diff, abstraction-leak answer). Your completion-report template must accommodate those additions — coordinate the exact format with `dev-1` (who owns T1 edits). If unclear, ask me.
- Don't add a "why B1 matters" paragraph. The role file is a prompt, not a sales pitch. State the rule. The reasoning goes in `dev-lessons.md` if anything.
- Worktree path for progress.md: you must decide between `.adaptive-team/progress.md` and some other location. I'd pick `.adaptive-team/` so future state files (e.g., the structured self-review, the pre-flight echo log) have a home. Add `.adaptive-team/` to `.gitignore` in the role file's "Worktree Setup" or equivalent section.
- Files < 500 lines; dev role file target < 150. Rules file target: do not exceed 250 lines (currently ~179).
- B6-twist introduces terminology: "universal checklist" vs "stack-specific checklist." Define it once, in the rules file, and reference it from all role files that use it.
- File ownership: you own `.claude/agents/adaptive-team-dev.md` and the relevant sections of `.claude/rules/adaptive-team-rules.md`. You do NOT own `.claude/agents/adaptive-team-sdet.md` — sdet is editing that in parallel for its own items. If an edit would touch sdet's file, message `sdet` first.
- **Run B1 on yourself**: after reading this briefing, SendMessage me a 3-bullet "what I understood" before touching any file. You are the first dev to practice the rule you're codifying.
- Do not merge to main until architect + sdet both return SATISFIED per the two-cycle gate.

# Architect Briefing — T4: PM + Rules + Skills + State Templates

**From:** `architect`
**To:** `dev-4`
**Task:** T4 — PM role, rules, skills, state templates
**Items touching design coherence / permissions / rules surface:** P2 (auto-consult on ambiguous), P9 (PM restates REASON ≤1 sentence; never opens DETAILS), P1-twist (event-driven heartbeat), P4 (rolling Recent Activity in `current-session.md`, no new file), P7 (reviewer-selection logged inline in Decisions), L2-contract (compaction-resilient identity block — first 30 lines of every agent file self-sufficient), P3 (reviewer-freshness audit every 3rd task), L8-rule (role Startup sections are *exhaustive* — every file listed with approximate size).

llm-expert is co-briefing you on the L2 and L8 parts (identity block and exhaustive Startup semantics). This briefing covers everything else plus how L2/L8 wire into the rules/PM-role surface.

## Read before you touch a file

1. `.claude/adaptive-team-learned/dev-lessons.md` — read first, always.
2. `.claude/agents/adaptive-team-product-manager.md` — the PM role file.
3. `.claude/rules/adaptive-team-rules.md` — the rules file.
4. `.claude/skills/adaptive-team-consult/SKILL.md`, `adaptive-team-implement/SKILL.md`, `adaptive-team-learning-moment/SKILL.md`, `adaptive-team-init/SKILL.md`.
5. `.claude/adaptive-team-state/current-session.md` — state template.
6. `.claude/adaptive-team-reviews/initial-audit/curious-synthesis.md` — the "P#" and "L#" items.

Do not read the rest of the codebase. Do not read other agent files except to verify a cross-reference (and in that case read only the referenced lines).

## The mentality

T4 is the **orchestrator's surface**. The PM is the one role whose failure silently corrupts everything else — because the PM is on the main thread, its drift cascades into compaction failures that poison the whole session. Every item in T4 is a mechanism that either (a) keeps the PM grounded, (b) makes PM behavior observable, or (c) narrows PM's room to improvise.

Two principles should bind your edits:

1. **Mechanism over exhortation.** The framework already has markdown rules telling the PM what to do. Those rules broke down on audit (see my audit findings C1, C2, C3 — "load-bearing rules live only in markdown"). Your edits should prefer *structural* enforcement: ordered Startup checklists, explicit triggers ("after every 3rd task"), file-level formats that a drifted PM cannot deviate from without violating an observable contract.

2. **Zero new files unless genuinely needed.** The synthesis converged hard on "fold into existing files, don't multiply state." P4 (no new idle-digest file), P5 (pending-pm-lessons staged in existing state dir), P7 (selection logged in Decisions, not a new audit file). Respect this — adding a new file for every new behavior is how the framework rots.

## What each item demands

### P2 — Auto-consult on ambiguous asks
In `adaptive-team-implement/SKILL.md` and the PM role file, add an intake check: if the user's task description cannot be restated as a single-sentence acceptance criterion, PM automatically routes to `/adaptive-team-consult` first and tells the user "I'm routing this to consult before implement because <the part that's unclear>." The test is *operational*: PM tries to write the one-sentence acceptance criterion, fails, therefore consult.

Do not make this a free-for-all escape hatch. Specify the trigger precisely: "if the task description contains an `or`, a question mark, or more than three distinct deliverables, PM must first confirm direction via consult." Architect's view: if a PM drifts into "I'll just implement something reasonable," we have lost the user's intent.

### P9 — PM restates REASON, never opens DETAILS
The audit's I3 finding (PM never reads DETAILS lets findings become a black box) is being partly resolved here. The rule is: PM's user-facing message on an UNSATISFIED verdict is a one-sentence restatement of the REASON line (the second line of the three-line verdict). PM does not open the DETAILS file. Period.

Add to the PM role file an explicit rule: "When relaying an UNSATISFIED verdict to the user, paraphrase the REASON in ≤1 sentence. You do not read DETAILS. The dev and reviewer own the substance; you route the signal." Add the same text to `adaptive-team-rules.md` under Verdict Format.

### P1-twist — Event-driven heartbeat
Today's "before going idle, report current local time" rule is right but under-specified. Make it comprehensive: PM emits a single status line on *every material event* — team-member spawned, team-member shutdown, task phase change, verdict received, decision logged, going idle. Format:

```
[HH:MM local | phase | task-id | arch=<✓|pending|✗> sdet=<…> | cycle N/2]
```

Define "material event" as: the events that already trigger a `current-session.md` update (the file already lists them). One status line per update. No timer-based polling (wastes turns).

### P4 — Rolling Recent Activity in current-session.md (no new file)
Do not create a `session-digest.md`. Add a `## Recent Activity` section to `current-session.md` that PM keeps to a rolling tail of the last N events (pick N=20). Every event emission (P1-twist) appends one line here. On compaction recovery, PM reads this section to reconstruct what happened since last compaction. The Recovery Protocol already has PM read `current-session.md`; you're making that read more informative.

### P7 — Reviewer-selection audit inline in Decisions
Today PM selects reviewers by signal from the Consult Mode rubric. Log the selection in `current-session.md`'s existing `## Decisions this session` section as a specific line format: `[signal: "<signal text>" → spawned: <reviewer-list>] — <task-id>`. Zero new files. At retrospective the user greps Decisions for reviewer-selection quality.

### L2-contract — Compaction-resilient identity block
Every agent role file's first ~30 lines must be *self-sufficient* for a cold-spawned agent: identity, tool palette / constraints, verdict format (if applicable). No cross-file references required to perform the role's most basic action. Today some role files front-load identity (PM, dev); others put it after startup lists. Standardize: frontmatter-like block → identity → tool palette (if applicable) → verdict format (if applicable) → everything else.

Write this as a **contract** in `adaptive-team-rules.md` under a new "Agent File Structure Contract" section. Role files themselves get a light audit — not your job to edit all six role files (that's wave 2), just codify the contract now.

### P3 — Reviewer-freshness audit every 3rd task
After every 3rd completed task in a session, PM sends each persistent reviewer: "Name your most recent concrete finding." Reviewer replies in one line. If the answer is vague or repeats earlier findings, PM triggers Reviewer Context Hygiene (spawn fresh reviewer with a handoff summary). Currently the hygiene rule is subjective; this makes it observable.

Add a counter to `current-session.md` (`Tasks completed this session: N`) and a PM trigger on `N % 3 == 0`.

### L8-rule — Exhaustive Startup sections
Every role file's `## Startup` section must list every file the role reads on spawn, with an approximate size indicator (`~60 lines`, `~200 lines`). llm-expert owns the rationale (budget is derivable from inspection, not from rotting advertised numbers). Your job: add the rule to `adaptive-team-rules.md` so future role-file edits must comply.

## Named failure mode (red-team clause)

Here is how a careless or rushed dev will get this task wrong: **they will add all eight items as new sections stacked at the bottom of `adaptive-team-rules.md`, leaving the existing `PM Context Discipline` (lines ~20-31), `Verdict Format` (lines ~76-84), and `Agent Timeout Protocol` (lines ~154-155) sections unchanged — producing a rules file where two different sections give two different instructions about the same thing (e.g., PM now has "restate REASON in ≤1 sentence" in the new P9 section AND "receives only verdicts (strict format above)" in the old Communication section, with no reconciliation).** Rules files must be *internally consistent*. Every edit is a chance to update the existing section rather than append a new one. If the existing `PM Context Discipline` section would contradict your new P9 rule, update that section — do not create a parallel one. When in doubt, collapse first, add second.

Additionally, these items must not inflate `adaptive-team-rules.md` past ~250 lines. Current is ~179. You have a ~70-line budget. If you exceed it, you're restating rather than editing.

## Gotchas

- **PM role file vs rules file duplication.** Several rules (Recovery Protocol, tool palette, verdict format) are stated in both `adaptive-team-product-manager.md` and `adaptive-team-rules.md`. Pick one as canonical (rules file is canonical), collapse the other to a pointer. Do this as part of T4 where it touches your items — otherwise the two files will drift from each other.
- **L2 identity block is a CONTRACT, not an edit of every role file.** You write the contract in `adaptive-team-rules.md`. You do NOT edit all six role files to comply — that's wave 2 (T5 "Final sweep"). Scope discipline: state the rule, don't rewrite six files.
- **P4 rolling activity tail** — define the truncation rule (last 20 events; older lines removed on append). Don't let the section grow unbounded — that defeats the purpose (PM re-reads this; an ever-growing tail kills the point).
- **P7 reviewer-selection log format** is a user contract — once you pick the exact format, every skill file that spawns reviewers must use it. Check `adaptive-team-consult/SKILL.md:22-33` and `adaptive-team-implement/SKILL.md:30-33` for where selection happens; update both to mention the logging.
- **P3 counter** — `Tasks completed this session` is a new field in `current-session.md`. Update the template. Also specify: "task completed" means reached ACCEPTED state, not just "dev reported completion." Ambiguity here creates a counter that increments on rejected work.
- **Files**: `adaptive-team-rules.md` ≤ 250 lines after edits. `adaptive-team-product-manager.md` ≤ 150 lines. SKILL.md files ≤ 120 lines each. `current-session.md` template ≤ 60 lines.
- **Worktree lock**: T4 and T2 both touch `adaptive-team-rules.md`. T2 is editing the Dev Startup Checklist section. T4 is editing almost everything else. Before any edit to the rules file, SendMessage `dev-2` (the T2 dev) with the exact line range you intend to modify, and confirm no overlap. If overlap, the dev who touches it second rebases on the other's merge.
- **llm-expert is co-briefing you** on L2 and L8. Read their briefing (`.claude/adaptive-team-reviews/impl-wave-1/T4/llm-expert-briefing.md` when it exists) before you write those rules. If it doesn't land before you start L2/L8, message `llm-expert` directly.
- **No new files outside the existing `.claude/adaptive-team-state/` directory.** If an item tempts you to create one, re-read the synthesis — someone already vetted it into "fold into existing file."
- **Run B1 on yourself**: SendMessage me a 3-bullet "what I understood" before touching any file. T4 is the heaviest task in the wave; echoing early catches the expensive misreads.
- Do not merge to main until architect + sdet (and llm-expert if it reviews any LLM-facing edit) both return SATISFIED per the two-cycle gate.

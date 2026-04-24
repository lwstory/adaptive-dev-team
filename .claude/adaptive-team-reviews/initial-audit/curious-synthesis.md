# Curious Synthesis — Initial Audit Ideation Pass

**Author:** `curious`
**Date:** 2026-04-23
**Inputs:** 60 ideas generated across 6 roles (architect, sdet, llm-expert, database, dev, pm), vetted by architect (its own + dev batches), sdet, llm-expert, database, and team-lead (PM batch).

Per-role sections below are structured **Keepers → Twists → Rejects** using the vetter's own words where quoted. Cross-role top-10 at the end.

---

## Architect — vetted by `architect`

### Keepers

- **A4. Dependency graph diff check** — Dev includes `deps-before.txt` / `deps-after.txt` in completion report; architect diffs it. *Vetter: "Cheap and high-signal. Unexpected deps are a classic smuggled-in-risk. Require justification in the completion report; architect rejects a verdict without it on dep-touching tasks."*

- **A5. Abstraction-leak test** — Dev names, in one sentence, what a caller of the new public API needs to know about its internals. *Vetter: "Sharp probe. Add to the dev completion report template; architect reads the answer, not the code, first."*

- **A7. Red-team briefing clause** — Every briefing ends with a named "here is how a careless/rushed dev would get this wrong" failure mode; dev must address it. *Vetter: "One named failure mode is the kind of concrete hook that shapes behavior. Missing address = auto UNSATISFIED."*

- **A10. Post-merge architectural debt log** — `.claude/adaptive-team-state/debt.md` captures accepted tech-debt with trigger conditions. *Vetter: "Right home. Critical: every entry needs a trigger condition for revisit (e.g., 'when tenant count > 50'), not a date. Dateless debt rots; trigger-based debt stays actionable."*

### Twists

- **A1. Gotchas sidecar →** Don't fork from the briefing; make it the final `## Gotchas` section of the briefing message itself. One artifact, no drift.

- **A2. Threat-model stub →** Keep for tasks touching auth / input boundaries / external I/O; skip pure refactors and internal plumbing. Binary "always STRIDE" adds ceremony where it isn't needed.

- **A6. Paired briefing with sdet →** Don't merge into one document (hides lens ownership). Require a 30-second async sync: architect drafts first, sdet appends or flags contradictions *before* the dev sees it. Preserves lens separation, kills the race.

- **A8. Permission-matrix artifact →** Make opt-in per project at init time. Only useful if the project's auth model is stable; for dynamic systems the matrix lives in code and the artifact rots.

- **A9. Briefing word budget →** 300 is too tight for multi-pitfall tasks. Cap at ~500 with a "principles first, then pitfalls, no code dumps" rule. Spirit right, number wrong.

### Rejects

- **A3. Dev-model selection as two-dimensional** — *Vetter: "Adds a knob for a gain I don't see. Sonnet vs Opus already implies context density in practice. A sentence-of-reason belongs in the briefing regardless; don't dress it as a new axis."*

---

## SDET — vetted by `sdet`

### Keepers

- **S1. Flake registry** — `.claude/adaptive-team-state/flakes.md` tracks intermittent failures; twice-flake = lesson. *Vetter: "Closes my audit's quarantine gap; twice-flake = lesson is the right threshold (once is noise)."*

- **S3. "Coverage hole" first test** — Sdet names the most-likely-missed behavior in briefing; if no test covers it, automatic UNSATISFIED. *Vetter: "Pre-committing the most-likely-missed behavior forecloses rationalized sign-offs. Keep the auto-UNSATISFIED strict."*

- **S6. Cold-start mode** — Full suite run against fresh DB + caches. *Vetter: "Best single catch for warmth-dependent flake. Tie to post-merge-on-main (deterministic), not 'periodically.'"*

- **S8. Briefing-as-test-plan** — Sdet's briefing IS the test plan, as an ordered list of behaviors; dev checks them off with behavior-IDs (not prose). *Vetter: "Structural fix for my audit finding C3 (briefings sidestep pyramid)."*

- **S10. Test-flake quarantine, not deletion** — Mandatory re-enable date + owning reviewer. *Vetter: "Right forcing function. Add: quarantine file itself fails the build after the date passes, so the deadline has teeth."*

### Twists

- **S2. Auto-draft layer plan →** Skeleton must be *named behaviors → layer*, not *test count → layer* (counts anchor on volume). Mark `DRAFT — critique required` so the dev doesn't treat it as final.

- **S4. Golden-set generator for LLM paths →** 5 is arbitrary; require coverage of the behavior axes named in the briefing. llm-expert owns golden-set design; sdet enforces existence, llm-expert vets quality.

- **S5. Test-mentality tags →** `contract` and `regression` overlap; collapse to `behavior | contract | characterization` plus optional `regresses: <issue-ref>`. Rejecting untagged tests is correct.

- **S7. Mutation test probe →** Full mutation testing is too heavy. Require only on paths sdet flags `critical` in the briefing. Manual "flip one condition, confirm a test fails" is the floor.

- **S9. Failure-mode matrix →** Valuable when task signals risk (auth, LLM, migration, concurrency); on trivial tasks it trains filler-noise tests. Make it triggered, not universal.

### Rejects
None.

---

## LLM-Expert — vetted by `llm-expert`

### Keepers

- **L2. Compaction-resilient identity block** — First 30 lines of every agent file are self-sufficient. *Vetter: "Frontmatter + first-30-lines-are-self-sufficient gives compaction-survival AND cold re-spawn survival. Worth writing as an explicit 'identity block' contract in adaptive-team-rules.md."*

- **L4. Prompt-injection red-team seed** — 3 adversarial inputs per external-text task. *Vetter: "Operationalizes the H11 trust-boundary concern. sdet + llm-expert pair here: sdet owns the test pattern, llm-expert owns the attack catalog."*

- **L6. Structured output contract per LLM call** — Pinned schema + parse-failure fallback. *Vetter: "Strongest pattern for LLM-touching product code and under-stated in the current role file. Add to the llm-expert universal checklist. 'We'll parse the text' = automatic UNSATISFIED."*

- **L9. Meta-eval suite — strongest of the ten** — Golden set of team scenarios, scored outputs. *Vetter: "This is what's missing from the framework as a whole: regression detection for framework changes themselves. Without it, audits are point-in-time; a golden-set runs continuously. llm-expert should own this — 'eval for the team' is our lens."*

- **L10. Shared vocabulary card** — 20 project terms with precise meanings. *Vetter: "Low cost, high value — especially when domain vocabulary is load-bearing ('tenant,' 'account,' 'workspace,' 'organization' are classic confusers). Preload via skills frontmatter so it hits every reviewer's context at startup without runtime reads."*

### Twists

- **L1. Agent-file cache map →** Harness caches are managed automatically; authored markers don't bind the harness. Twist: enforce a canonical top-of-file structure across all role files (frontmatter → identity → tool palette → verdict format). Same cache-hit benefit, honest about what we can guarantee.

- **L3. Dev model auto-downgrade →** Token count isn't correlated with task complexity, and Haiku's reasoning ceiling is lower regardless. Twist: architect picks from `haiku | sonnet | opus` by judgment. Document Haiku as appropriate for mechanical tasks (rename, move file, trivial test). Harness already supports `haiku` alias.

- **L8. Context-budget advertised per role →** Advertised numbers rot fast and become decorative. Twist: each role file's Startup section is *exhaustive* — list every file read at startup with approximate size, so budget is derivable from inspection.

### Rejects

- **L5. Briefing style linter** — *Vetter: "Linters on creative prose produce goodhart effects — reviewers write to the linter, not to the dev. Briefing style is enforced socially and by the agent file. A hook flagging token-budget overruns is fine; word counts and checklist caps are too prescriptive."*

- **L7. Agent-file A/B with -v2.md** — *Vetter: "Git branches do this better. Experimental agent-file changes belong on a branch; user opts in by checkout. In-repo versioning creates 'which file is live?' confusion, invites drift, and conflicts with /adaptive-team-init's overwrite model."*

---

## Database — vetted by `database`

### Keepers

- **D1. Query plan artifact** — Dev includes EXPLAIN/PROFILE in completion report on schema/query work. *Vetter: "Cheap for dev, high signal. 'Tests pass' hides planner regressions."*

- **D2. Migration two-phase checklist** — Expand / backfill / contract, required briefing artifact. *Vetter: "Already implicit in my briefing style; promoting it to a required briefing artifact makes the discipline enforceable rather than reviewer-remembered."*

- **D4. Cross-store consistency audit** — Idempotency + retry plan, or auto-BLOCKED. *Vetter: "Most common silent-data-loss vector I see. Auto-BLOCK on 'just hope' is the right hammer. Extend 'two stores' to include 'store + external side effect' (e.g., MySQL + webhook)."*

- **D9. PII routing rule** — Mandatory PII classification per new field. *Vetter: "Forces retention/deletion thinking at field introduction. Sharpen enum to `none / internal / PII / sensitive-PII` — 'low/high' is fuzzy."*

### Twists

- **D3. Synthetic-load eyebrow →** Blanket scaled fixtures per-task is too heavy. Instead: require the dev to state expected production cardinality and justify the index choice against it; reserve the scaled-fixture demand for tasks where the justification is thin.

- **D6. Team telemetry store →** Good instinct, wrong scope. Append JSONL to `.claude/adaptive-team-state/telemetry.jsonl`, query with `jq`. SQLite is over-engineering for an append-only event log; upgrade only if queries become repetitive.

- **D8. Schema freeze in worktree →** Concern correct, soft-lock weak. Tighten: PM refuses to spawn `dev-2` with `schema-touching: yes` if any current dev is also schema-touching. Serialize at the orchestration layer, not via advisory file.

- **D10. Data-boundary diagram per story →** Mandating per story is heavy. Require only for stories touching ≥2 stores or crossing a trust boundary. Single-store stories get a one-line data-flow note, not a diagram.

### Rejects

- **D5. Lesson file durable-state (SQLite)** — *Vetter: "Premature. Markdown is the right substrate for HITL approval + git blame. SQLite breaks diff review, breaks user-approval workflow, trades a real benefit (searchable markdown in PR review) for a hypothetical one (query by tag). Revisit at 200+ lessons. Dedup is the actual need; grep solves it."*

- **D7. Graph of lessons (Neo4j)** — *Vetter: "Solution looking for a problem. Contradictory-lessons is real but the cause is absence of pre-propose dedup, not absence of a graph. Neo4j for ~50 nodes is absurd; grep + user review during approval handles conflict detection at this scale."*

*Database's standing observation: both rejects "substitute infra for process — when the real gap is the process step (dedup, cardinality justification), adding a database doesn't fix it."*

---

## Dev — vetted by `architect` (owns dev briefings)

### Keepers

- **B1. Pre-flight echo** — Dev SendMessages a 3-bullet "what I understood" back to architect before touching code. *Vetter: "Right half of C2 (briefing race) from my audit. Cheap, catches misread briefings in minutes. Add to dev startup Step 4."*

- **B4. Test-first toggle per task** — Selective TDD. *Vetter: "Let sdet flag the toggle, not architect (test-strategy call). When set, dev commits failing tests first; sdet reviews the test commit before implementation."*

- **B5. Structured progress.md template** — Fixed Decisions / Blockers / Next sections. *Vetter: "Beats free-form for re-spawn recovery. Add Intent as a fourth section ('what this task is trying to achieve in one line')."*

- **B8. Unknown-unknowns ask** — Dev messages architect with its top uncertainty. *Vetter: "Forcing one uncertainty question is a strong debiasing move — devs default to confidence. Pairs with B1: echo first, then the one question."*

### Twists

- **B2. Worktree bootstrap script →** Shell script is brittle. Make it declarative: `worktree-setup.md` with steps, dev executes and reports. A script can optionally exist alongside but isn't the source of truth.

- **B3. Explicit no-surprise file lock →** Half-exists via File Ownership in rules. Don't add a second manifest; require the dev's completion report to list *touched* files and flag any outside the owned-files list. Scope drift surfaces at review, not via unbuilt automation.

- **B6. Self-review pass before PM report →** Narrow it: dev runs universal checklists only (not stack-specific), one line per item (pass/fail/NA with note). Reject free-form `dev-self-review.md`; accept a bounded checklist response.

- **B9. Commit-chunk plan →** Keeper for multi-file tasks; overkill for single-file fixes. Require only when task touches ≥3 files or has estimated diff > ~150 lines. Architect can override on spawn.

### Rejects

- **B7. Time-to-first-test metric** — *Vetter: "Metric for metric's sake. Single-task wall-clock data is noisy, and reviewers already spot scaffolding-heavy tasks from the completion report. Don't instrument what review reveals."*

- **B10. Diff-budget warning** — *Vetter: "50/100/150% check-ins add synchronous friction and assume good up-front estimation. Scope creep is better caught by B3 (file-lock), 2-cycle review cap, and B1 (pre-flight echo). Three mechanisms already; don't add a fourth that interrupts flow."*

*Architect's priority keepers that compound: A4, A7, A10, B1, B5, B8 — these plug specific gaps from the audit (C2, I1, doc drift).*

---

## PM — vetted by `team-lead` (PM on main thread)

### Keepers

- **P2. Auto-consult first on ambiguous asks** — Force route to `/adaptive-team-consult` when acceptance criteria won't fit in one sentence. *Vetter: "Concrete, testable, prevents rushing to implement on fuzzy asks. Add to PM role file and implement skill intake."*

- **P3. Reviewer freshness audit** — After every 3rd task, PM asks each persistent reviewer to name its most recent concrete finding. *Vetter: "Existing 'reviewer context hygiene' rule is subjective. 'Ask each reviewer after every 3rd task to name its most recent concrete finding' is observable and deterministic. Adopt as-is."*

- **P10. Kill-switch budget** — Token/time budget with 80% warn, 100% confirm. *Vetter: "Ambiguous asks can run away today; cheap guardrail. Add `budget:` field to session state so it survives compaction."*

### Twists

- **P1. Session heartbeat →** Timer-based 5-min heartbeats are noise when nothing's happening and overlap during activity. Make it *event-driven*: PM emits the status line on every material event (spawn, verdict, phase change, going idle). Same signal, no wasted turns.

- **P4. Session digest on idle →** Don't add a new file. Append a rolling "Recent Activity" tail to `current-session.md` (last N events). One file to keep current; survives compaction; next session reads it first by existing Recovery Protocol.

- **P5. User-adaptation journal →** Don't create a separate `user-style.md`. Use `.claude/adaptive-team-state/pending-pm-lessons.md` as a staging file: PM writes observations in-flight; at session boundary user reviews and approves → promotes to `pm-lessons.md`. Also solves the database audit's "no transaction boundary for lesson propose→approve→write" by persisting proposals durably.

- **P6. Structured "one-liner" verdict for user →** Don't replace prose with a robotic line; *append* it as a footer: `[phase | task | arch=✓ sdet=pending | cycle 1/2]`. Prose + footer = readable for humans, scannable when many turns accumulate.

- **P7. Reviewer-selection audit →** Don't create a separate audit file. Log the selection in `current-session.md`'s Decisions section as `[signal: "schema, migration" → spawned: database]`. At retrospective, natural grep. Zero new files.

- **P8. Task-risk tag at intake →** "Green skips paired briefings" undermines shift-left. Keep risk tags to influence *which* reviewers spawn and briefing depth, but never eliminate briefings entirely. Green = architect briefing only; yellow = architect + sdet; red = all relevant.

- **P9. PM doesn't route verdicts verbatim →** Keeper with a constraint: PM restates the blocker in ≤1 sentence using the REASON line only. PM does NOT open DETAILS. Adds comprehension to the user-facing relay without violating context discipline.

### Rejects
None outright. *Vetter observation: "Many twists fold back into existing state files rather than multiplying artifact types."*

---

## Editor's Top-10 Cross-Role Picks

Ranked by vetter enthusiasm + compounding effect across roles + implementation cheapness.

| # | Idea | Role | Why it's top-10 |
|---|------|------|----------------|
| 1 | **L9. Meta-eval suite** | llm-expert | "Strongest of the ten" per llm-expert. Only idea that turns point-in-time audits into continuous regression detection *for the framework itself*. Highest strategic value. |
| 2 | **B1. Pre-flight echo + B8. Unknown-unknowns ask (pair)** | dev | Architect flagged as plugging audit finding C2 (briefing race). Cheap, compounding — echo confirms understanding; one question surfaces the biggest risk. |
| 3 | **S3. "Coverage hole" first test** | sdet | Pre-commits the most-likely-missed behavior *before* review. Forecloses rationalized sign-offs. Structural, not ceremonial. |
| 4 | **A7. Red-team briefing clause** | architect | One named failure mode per briefing; dev addresses it in completion report. Concrete hook that changes behavior without extra artifact. |
| 5 | **L6. Structured output contract** | llm-expert | "Biggest product-code win." Pinned schema + parse-failure fallback = most LLM-call failure modes covered. Belongs in universal checklist. |
| 6 | **D2. Migration two-phase checklist** | database | Promotes discipline from "reviewer-remembered" to "briefing-required" for every migration. Cheap, prevents real production incidents. |
| 7 | **S8. Briefing-as-test-plan** | sdet | Closes audit finding C3 (briefings sidestep pyramid). Dev completion report references behavior-IDs not prose — auditable. |
| 8 | **A10. Post-merge architectural debt log** | architect | Makes tech-debt explicit with **trigger conditions**, not dates. Simple state file, long half-life. |
| 9 | **L10. Shared vocabulary card** | llm-expert | Load-bearing domain terms ("tenant," "account," "workspace") confuse quietly. Preload via skills frontmatter — cheap, preempts a class of error. |
| 10 | **P2. Auto-consult on ambiguous asks** | pm | PM gate that prevents rushing to implement on fuzzy asks. One-sentence acceptance-criteria test is operationally testable. |

### Honorable mentions (runners-up worth noting)

- **A4. Dependency graph diff check** — high-signal on dep-touching tasks; would be in top-10 if scoped tighter.
- **L2. Compaction-resilient identity block** — structural prompt-engineering win for the agent files themselves.
- **D4. Cross-store consistency audit** — "most common silent-data-loss vector" per database.
- **P3. Reviewer freshness audit** — turns a subjective rule into an observable check.
- **S1. Flake registry** — closes sdet's audit quarantine gap.

### Common threads across roles

1. **Briefing rigor compounds.** Red-team clauses (A7), briefing-as-test-plan (S8), pre-flight echo (B1), unknown-unknowns (B8), vocabulary card (L10) all point at the same insight: the highest-leverage fixes happen *before* code is written.
2. **Artifact minimalism.** Vetters repeatedly rejected "new file" proposals in favor of folding into existing files (P4, P5, P7, A1 gotchas into briefing). Resist multiplying state files.
3. **Triggered, not universal.** Threat-model stubs (A2), failure-mode matrix (S9), diagrams (D10), commit-chunk plans (B9) — all became keepers only when scoped to tasks with the relevant signal. Mandatory-everywhere ceremony breeds filler.
4. **Infra is not process.** Database's two rejects (SQLite lessons, Neo4j graph) and llm-expert's reject of A/B files all flagged the same antipattern: adding infrastructure to cover what should be a process discipline.
5. **Framework-on-framework.** L9 meta-eval suite is the one idea multiple vetters implicitly endorse by analogy — it's what would let us vet *future* ideation passes against a baseline rather than via one-off audits.

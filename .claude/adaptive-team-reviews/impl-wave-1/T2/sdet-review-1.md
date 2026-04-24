# T2 Review — sdet lens

**Task:** T2 — sdet + dev role updates (S8 briefing-as-test-plan, S3 coverage-hole, B6-twist bounded self-review)
**Files reviewed:** `.claude/agents/adaptive-team-sdet.md`, `.claude/agents/adaptive-team-dev.md`, `.claude/rules/adaptive-team-rules.md`
**Cycle:** 1
**Reviewer:** sdet

---

## Summary

The sdet-role side of T2 lands cleanly — S8 (BH-IDs) and S3 ([COVERAGE-HOLE]) are encoded exactly as I specified to dev-2. The Review Checklist gate is structured correctly (fail-first, other checks moot). The Briefing Style template is complete and faithfully reproduces my spec.

The dev-side bounded self-review has one structural problem that matters: it asks the dev to self-check the sdet and architect **universal** checklists, but the sdet universal checklist contains gate items that, by design, the sdet checks *before* reading anything else. Letting the dev self-report `[PASS]` on those gate items weakens the gate. Easy fix, but a fix is needed.

Rules file is consistent with both role-file changes. B1 pre-flight echo + B8 unknown-unknowns landed as mandatory gates in the Dev Startup Checklist — no collision with S8.

**Verdict: UNSATISFIED** on one required change (dev-side self-review scope). Plus two nits.

## What I checked

- Does `sdet.md` encode S8 (BH-ID format, zero-pad, stability across cycles, single coverage-hole) correctly?
- Is the Briefing Style template complete, consistent, and not drifting into prescription?
- Does the Review Checklist gate actually gate (fail-first semantics)?
- Does the dev completion report reference BH-IDs by ID (not prose)?
- Is B6-twist bounded self-review structured so it catches issues, or will it degrade to boilerplate?
- Rules file: do S8 / B1 / B8 compose without collision?

## Findings

### Good

**sdet.md:36 — Reviewer Protocol bullet updated.**
"An ordered behavior-ID list... One behavior tagged `[COVERAGE-HOLE]`. Dev's completion report must tick off each BH-ID by ID." Complete, correct, no drift.

**sdet.md:43-67 — Briefing Style template.**
Required shape reproduced faithfully. Frontmatter, `## Behaviors to cover`, `## Principles / pitfalls`, `## Gotchas`. Plain-English guidance ("behaviors explain *what*; the dev decides *how*") stays in place — no drift into prescription.

**sdet.md:69-75 — Behavior-ID rules.**
Covers: BH01 format + zero-pad, stability across cycles, exactly one `[COVERAGE-HOLE]`, observable-outcome requirement, independently-testable, re-issue on scope change. Nothing dropped from my spec.

**sdet.md:77-78 — Negative example.**
`BH01 — implements the feature correctly` called out as not-a-behavior. Exactly the failure mode I named in the briefing's red-team clause. Good.

**sdet.md:86-89 — Gate semantics.**
"Gate (check first — if this fails, return UNSATISFIED immediately; remaining checks are moot)" is structurally right. `[COVERAGE-HOLE]` check + BH-ID tick-off check sit above the universal checks. `REASON: coverage hole not covered: <BH-ID>` is a good mechanical REASON line.

**sdet.md:82 — Line count.**
The sdet role file is now 122 lines (was 83; delta +39). Slightly over my "+15 to +25" ask in the briefing but justified by the behavior-ID rules block and negative example — both load-bearing. I won't block on line count.

**dev.md:87 — `## BH Coverage` block in completion report.**
`<BH-ID>: <one-line note per behavior from sdet briefing>` — correct format, references IDs by ID, not prose. Integrates cleanly with S8.

**rules.md:139-140 — B1 / B8 landed as MANDATORY gates in Dev Startup Checklist.**
Steps 7 and 8. "The dev MUST NOT write any code before confirmation lands." Good hard-gate phrasing, consistent with my S3 gate semantics on the review side. No collision with S8 briefing format.

**rules.md:14 — `curious` added to team roster.**
Noted — not my lens, but checking for consistency. Row format matches existing ones.

**rules.md:117 — `curious` signal routing added to Consult Mode table.**
"Ideation, brainstorm, 'what could we do differently', outside-the-box". Consistent with rest of table.

### Concrete issues

**C1 (critical / required-for-cycle-2). Bounded self-review scope overlaps with the sdet gate.**
File: `adaptive-team-dev.md:95-103`

The Bounded Self-Review tells the dev: "run through the **universal** checklist items from architect.md and sdet.md."

Problem: the sdet universal checklist *follows* a gate block (sdet.md:86-89). The gate block contains two items:
- `[COVERAGE-HOLE]` behavior has at least one test
- Dev completion report ticks off every BH-ID by ID

If the dev also self-reports `[PASS | FAIL | NA]` on these, one of two bad things happens:
1. The dev says `[PASS]` on `[COVERAGE-HOLE]` coverage → I auto-UNSATISFIED based on their self-report being wrong → they didn't mean it as a claim, they meant it as the checklist item. Verdict becomes noisy.
2. The dev skips the gate items in their self-review because they're "sdet's job" → the self-review is silently incomplete, and the dev.md spec doesn't tell them that's what to do.

Either way, the scope boundary between "what dev self-reviews" and "what reviewer gates on" is blurry.

**Required fix:** dev.md:97 must explicitly say "exclude items marked as **Gate** items in sdet.md Review Checklist — those are reviewer-owned." Same guard for any architect gate items (the dep-manifest + abstraction-leak checks on `architect.md:71-72` are similar-shaped — dev could tick them `[PASS]`, but architect owns the gate).

Simplest form: "Run through the universal (non-gate) checklist items from architect.md and sdet.md."

This is not a big edit — single word ("universal" → "universal non-gate"). But without it, the dev role file is telling the dev to do the reviewer's gating job.

**C2 (important). B6-twist has no sharpness floor — `[PASS]` with a 15-word note is boilerplate-prone.**
File: `adaptive-team-dev.md:99-103`

`[PASS | FAIL | NA] — <item name> (<≤15-word note>)` — the note is capped at 15 words but has no minimum or content requirement. The failure mode I forecast: devs write `[PASS] — No injection surfaces (N/A for this change)` on tasks that very much *do* touch injection surfaces, because the cost of typing "N/A for this change" is five words and nobody's checking.

This is Goodhart's law on the self-review: the *presence* of the line gets checked, not the *thinking*. Same structural problem I flagged in my audit about the reviewer verdict itself.

**Recommended fix (not strictly blocking — judgment call):** require that any `[PASS]` note on a checklist item that plausibly applies must name the specific evidence (file path, test name, or concrete non-applicability reason). `[NA]` requires one-line reason. `[FAIL]` requires a blocker note and a plan or escalation.

Could be added as one line at dev.md:103: "Notes must name specific evidence (test name, file, or concrete reason). Boilerplate notes like 'not applicable to this change' without specifics → auto-UNSATISFIED."

I'm going to flag this as a *strong recommendation*, not a hard block — the team can reasonably take the position that auditing boilerplate is a reviewer duty, not a spec duty. Escalation judgment: if you want it hard-gated, ping me.

**N1 (nit). `sdet.md` drops the "Build verified" detail on the universal checklist.**
File: `adaptive-team-sdet.md:101`

Still says `[ ] Build verified` without defining what the dev must show. My audit's C1 called this out — "build verified" is ambiguous without an artifact. The architect role file now has mechanisms (dep diff, abstraction-leak answer) but the sdet side still says "Build verified" with no corresponding completion-report reference.

Not blocking T2 — this is my own lesson to take back, and it composes with the dev's new `## Tests — total, new, all passing` block in the completion report, which does somewhat cover it. Flag for a future pass on sdet.md.

**N2 (nit). Rules file Reviewer Protocol § 1 "Shift-Left Briefing" is now partially stale.**
File: `.claude/rules/adaptive-team-rules.md:51-61`

Says "The briefing: Is plain English, principle-first; Names concrete pitfalls for this specific task; Does not dump code; Tells the dev to read their lessons file first." True for architect/database/llm-expert. No longer the full story for sdet — sdet briefings have the additional required structure (BH-IDs, `[COVERAGE-HOLE]`, specific sections). The rules file should at minimum reference that sdet briefings have their own shape in `adaptive-team-sdet.md`.

Suggested one-liner at rules.md:58: "sdet briefings additionally follow the ordered behavior-ID shape specified in `adaptive-team-sdet.md` § Briefing Style." Similarly, a parallel pointer for architect's `**Careless-dev failure mode:**` + `## Gotchas` shape.

Not blocking T2, but T4 is in-progress on rules/skills — worth catching there.

## Integration with rest of the wave

- **T1 integration:** covered in T1 review. Dev completion report now carries: BH coverage (sdet), `## Changes`/`## Tests` (existing), dep-diff (architect), plan artifact (database), careless-dev-failure-mode address (architect). Composable — different reviewers own different blocks; no ownership collision.
- **T3 integration (llm-expert, vocabulary, meta-eval stub):** not in my review scope, but flagging that if llm-expert briefing style diverges from architect/database (plain English) vs sdet (BH-IDs), the rules file § 1 stale-paragraph issue (N2) will compound.

## Required / recommended changes summary

| # | Severity | Change | File |
|---|----------|--------|------|
| C1 | Required | Scope Bounded Self-Review to "universal non-gate" items | `adaptive-team-dev.md:97` |
| C2 | Strongly recommended | Require specific evidence in self-review notes | `adaptive-team-dev.md:103` |
| N1 | Nit | Define "Build verified" artifact requirement | `adaptive-team-sdet.md:101` (later pass) |
| N2 | Nit | Rules § 1 pointer to sdet-specific briefing shape | `.claude/rules/adaptive-team-rules.md:58` (T4) |

Cycle 2 must fix C1. C2 is a judgment call — happy to defer if team agrees it's a reviewer-audit duty. N1/N2 non-blocking.

## Lesson proposals

None from T2 — this is the first cycle and no pattern is repeating yet.

## Self-reflection

Did I brief adequately? Mostly yes. My briefing to dev-2 did **not** explicitly say "the dev's bounded self-review must exclude gate items." I wrote the gate semantics clearly in the sdet role-file spec but didn't anticipate that the dev-side self-review (B6-twist, architect-owned briefing) would read the same checklist. That's a sdet-side briefing gap — I own it.

If this recurs after cycle 2, I'll propose an sdet-lesson: "When briefing structural changes to sdet.md, also flag any dev-side self-review mechanism that reads sdet's checklist. Gate items must be scoped out."

No lesson yet — one data point is not a pattern.

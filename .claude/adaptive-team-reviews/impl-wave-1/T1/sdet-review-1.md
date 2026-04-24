# T1 Review — sdet lens

**Task:** T1 — architect + database role updates (A5 abstraction-leak probe, A4 dep diff, database plan artifact)
**Files reviewed:** `.claude/agents/adaptive-team-architect.md`, `.claude/agents/adaptive-team-database.md`
**Cycle:** 1
**Reviewer:** sdet

---

## Summary

T1 lands well from my lens. The architect's dep-manifest gate and abstraction-leak pre-read materially strengthen what the architect can actually catch *before* it depends on my coverage pass. The database plan artifact is exactly the kind of "trust but verify" hook I wanted to see at the architect-review tier generalized to the data layer. No blockers. Two concrete tightening asks below.

## What I checked

- Does the architect's completion-report protocol give sdet enough structural hook to gate tests properly?
- Does it integrate cleanly with my S8 briefing-as-test-plan (BH-IDs) pattern, or does it collide?
- Is the database plan-artifact check gated hard enough to actually fire, or is it advisory?

## Findings

### Good

**architect.md:40 — Red-team clause is grep-detectable.**
The fixed label `**Careless-dev failure mode:**` makes the check mechanical: `grep -L` finds briefings missing the label. A completion report that doesn't address it is auto-UNSATISFIED. This is the right shape — doesn't rely on reviewer judgment to notice an absence.

**architect.md:71 — Dep manifest diff in completion report.**
Absence → UNSATISFIED is stated hard, not advisory. Good. This closes a real gap: devs silently pulling in a transitive dep that broadens the test surface (new security scanning required, new flake modes, new licensing) without anyone noticing.

**architect.md:72 — Abstraction-leak pre-read.**
"Read the dev's answer *first*, before reading code" is a nice cognitive-ordering trick: it forces the architect to form a hypothesis from a prose description before being primed by the code. Good defense against rubber-stamping.

**database.md:43 — Plan artifact gate.**
Hard "absence → UNSATISFIED" with engine-specific commands (`EXPLAIN (ANALYZE, BUFFERS)`, `PROFILE`, `.explain("executionStats")`). Exactly the kind of observable artifact my audit asked for — I can point at the file, not trust a verdict.

**database.md:44 — Baseline diffing.**
"No baseline on file is not an excuse to skip — capture it now" closes the usual escape hatch. Good.

### Integration with my S8 briefing (BH-IDs)

No collision. Architect's briefing shape ends with `**Careless-dev failure mode:**` + `## Gotchas`. My sdet briefing shape has `## Behaviors to cover` + `## Principles / pitfalls` + `## Gotchas`. Both can coexist because they're produced *by different reviewers* and the dev reads both.

**One concern worth calling out:** the dev's completion report now has to satisfy three orthogonal structural demands — (a) tick off BH-IDs (sdet), (b) address `**Careless-dev failure mode:**` (architect), (c) include plan artifact + dep diff (architect + database). The dev-side completion-report section (`adaptive-team-dev.md:80-93`) does have the `## BH Coverage` block, which is good. But it currently does **not** have an explicit block for "Careless-dev failure mode addressed." That's an architect-owned gap, not mine, but flagging it because it affects how the gate composes.

### Concrete issues

**I1 (important). Database plan-artifact gate is keyed to completion-report presence, not content quality.**
File: `adaptive-team-database.md:43-44`

The check is "Completion report includes a plan artifact." That fires on *any* `EXPLAIN` output being present. A dev can paste a plan for the wrong query (the one that's *not* changed), or a plan captured on an empty table, and the gate passes. My lens cares because "plan artifact attached" will become the thing devs optimize for, not "plan artifact for the query that changed."

**Recommendation:** Tighten to "plan artifact for the query-ID(s) named in the briefing's behavior list or in the dev's diff." Ties the artifact to a specific query, so pasting a stale/wrong plan is catchable. This is a database-owned change, not mine to edit — passing the signal.

**I2 (nit). Database Review Checklist is missing "test-data density matches production shape" for plan artifacts.**
File: `adaptive-team-database.md:32-44`

A plan captured against a dev fixture with 100 rows says nothing about the hot-table case my audit brief flagged. This isn't blocking T1, but for the plan gate to do real work, the baseline (or the single captured plan) needs a note on the row-count / cardinality it was run against. Otherwise `EXPLAIN` on a toy dataset will rubber-stamp queries that fall over in production — exactly the failure mode the check was built to prevent.

**Recommendation:** Add to database.md:44: "Plan capture must note approximate row-count / cardinality of the table(s) involved. Plans captured against <1k rows for tables that exceed that in production are flagged and re-run against a representative dataset."

### Briefing-as-test-plan compatibility

**S8 compatibility check: PASS.**

- Architect briefings are *not* the behavior list — they're principle + pitfalls + red-team + gotchas. Good; lanes stay separate.
- Dev's completion report has a dedicated `## BH Coverage` block (`adaptive-team-dev.md:87`) so my BH-IDs don't get lost in the prose.
- Architect's dep-diff and abstraction-leak artifact don't require BH-IDs, which is right — those are architectural concerns, not behavioral ones.

**One subtle hazard:** if a dep change *itself* introduces new behavior (e.g., adding a JWT library changes token-validation behavior), that behavior must land in my briefing as a BH-ID. Today nothing in the protocol flags "dep change implies new behavior to test." Not a T1 defect — just noting for a future cross-file lesson if it surfaces.

## Verdict rationale

Structurally sound, pulls strongly in the direction my audit asked for. The two tightening asks (I1 content-vs-presence gating, I2 cardinality note) are improvements, not blockers. I could see a case for SATISFIED-with-nits here, but I want the I1 fix in cycle 2 because the plan-artifact check's whole value is that it's verifiable — and "any plan, any query" is not verifiable in the way I'd want.

**Required change for cycle 2:** address I1 on database.md (tie plan artifact to named query-ID in briefing or diff). I2 is a nit — would be nice, not required.

## Lesson proposals

None from my lens on T1. The protocol is sound; the tweaks are refinements, not lessons.

## Self-reflection

Did I brief adequately? Briefing covered the SDET side of T2, not T1 — architect owned the briefing here. No sdet-briefing inadequacy to own. I1 is a potential architect-or-database briefing gap (absence-vs-content gating), but I'd want to hear their self-reflection before assigning.

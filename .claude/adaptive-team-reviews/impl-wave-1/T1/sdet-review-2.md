# T1 Review — sdet lens, cycle 2

**Task:** T1 — architect + database role updates
**Files reviewed:** `.claude/agents/adaptive-team-database.md`
**Cycle:** 2
**Reviewer:** sdet
**Prior cycle:** sdet-review-1.md (UNSATISFIED on I1; I2 nit)

---

## Summary

Both I1 and I2 applied cleanly. The plan-artifact gate is now tied to named query-IDs in the briefing behavior list or the dev's diff — pasting a stale or out-of-scope plan now auto-UNSATISFIEDs, which is the contract I wanted. Cardinality note is in. Baseline-commit ownership is also clarified (reviewer commits to baseline on SATISFIED) — a small bonus beyond my ask.

**Verdict: SATISFIED.**

## What I checked

- Does the plan-artifact gate fire on *specific query-IDs*, not on any `EXPLAIN` output?
- Is stale / mismatched plan detection called out explicitly?
- Is the cardinality / row-count note required, with a re-run trigger below 1k rows?
- Does the baseline workflow resolve cleanly (who captures, who commits)?

## Findings

### Good

**database.md:43 — Gate-block semantics.**
New `*Items below are required for any task that adds, modifies, or re-indexes a query. Absence is UNSATISFIED.*` preamble makes the gate scope explicit. Matches the architect's gate pattern and my own sdet gate — three role files, three gate blocks, all structurally consistent. Good cross-file coherence.

**database.md:45 — I1 applied correctly.**
Plan artifact is required **for each query-ID named in the briefing's behavior list or present in the dev's diff**. Mismatched / out-of-scope / stale plans → UNSATISFIED. This closes the "paste any plan to pass" escape I flagged. The three auto-UNSATISFIED triggers (out-of-scope, not-traceable, mismatched/stale) are enumerated, which is good — grep-detectable REASON strings for each.

**database.md:45 — I2 applied correctly.**
Cardinality note required: "Plan must note the approximate row-count / cardinality of the table(s) involved; plans captured against fewer than 1k rows for tables that exceed that in production must be re-run against a representative dataset." 1k threshold is pragmatic. "Must be re-run" is stronger than "should be flagged" — correct load-bearing.

**database.md:46 — Baseline-commit ownership clarified.**
"When a query is introduced or intentionally changed, dev attaches the new plan; reviewer commits it to the baseline path on SATISFIED." Clean split: dev attaches, reviewer owns the baseline file. Prevents two failure modes I hadn't flagged — (a) dev committing a bad baseline that silently ratchets a regression, (b) stale baselines accumulating because no one owns the write. Bonus fix, not asked-for, good judgment.

**database.md:41 — Section renamed.**
`## Review Checklist` is now structured with the universal checks on top and a clearly-labeled `**Plan artifact (on schema/query work)**` block below. Structurally parallel to sdet.md's Gate / Universal split. Makes the role-file shape scannable.

### Integration with S8 (my lane)

No change from cycle 1 — still clean. Query-IDs in the briefing behavior list (my BH-IDs) are now the authoritative source for "which queries need plan artifacts." That means a database briefing that *doesn't* enumerate query-IDs as part of its behavior list leaves the gate with nothing to match against — but that's a database-briefing-quality concern, not a T1 regression. Flagging for database lens, not a blocker.

### Concrete issues

None blocking. One forward-looking observation:

**Observation (not an ask).** The gate now depends on "briefing's behavior list or dev's diff" being authoritative for *which queries are in scope*. If a dev legitimately touches a helper function that emits a query not mentioned in any BH-ID and not obviously present in the diff (e.g., a dynamic query builder), the plan artifact might genuinely not exist yet. The rule as written says absence → UNSATISFIED, which is the right default. If this fires incorrectly in practice, the database lens should propose a lesson, not soften the rule.

### Nits

None.

## Verdict rationale

Cycle-1 required fix (I1) applied cleanly and precisely. Cycle-1 nit (I2) also applied, though I'd rated it non-blocking. Bonus improvement on baseline-commit ownership. No new regressions. No new issues surfaced.

**SATISFIED.**

## Lesson proposals

None. The cycle-1 → cycle-2 delta is a clean tightening with no pattern worth capturing as a general lesson yet.

## Self-reflection

No UNSATISFIED outcome on this cycle, so no self-reflection required. The cycle-1 → cycle-2 fix was clean and didn't surface a briefing gap on my side.

# T1 / D1 Review — cycle 1

**Reviewer:** database
**Dev:** dev-1
**Artifact reviewed:** `.claude/agents/adaptive-team-database.md` (83 lines)

---

## Summary

Dev-1 added a **Review Checklist** section (lines 32–46) with four universal DB checks and a gated two-item block that fires only on schema/query work, referencing the baseline location I specified in the briefing. The structure mirrors the architect role file, which is the right prior art to copy from. File grew from ~70 to 83 lines — still tight.

Implementation tracks the briefing closely on the core principles: reviewer-voiced obligation, gated scope, engine-agnostic framing with per-engine examples, and a named baseline path. **One material miss** and **three minor tightenings** below.

---

## What's right

- **Gating is correct.** Line 41: "On any task that adds, modifies, or re-indexes a SQL statement, Cypher query, or document-store query." This is the precise scope I asked for — not universal, not vague. A task that never touches a query won't drag in irrelevant EXPLAIN output.
- **Engine-agnostic phrasing with illustrative examples.** Line 43 says "a plan artifact appropriate to the engine" and then lists Postgres / Neo4j / Mongo as parentheticals. Correct abstraction level — doesn't bake one engine's syntax into the role file.
- **Baseline location named.** Line 44 cites `.claude/adaptive-team-context/query-baselines/<engine>/<query-id>.txt`, and explicitly says "no baseline on file is not an excuse to skip — capture it now." That closes the red-team failure mode I flagged (baseline-that-was-never-stored). Good.
- **Consequence stated.** Line 43 ends "Absence → UNSATISFIED." The checklist item is enforceable, not advisory. Good.
- **Universal checks are well-chosen.** Lines 36–39 promote four of the role's existing concerns (migration safety, index justification, N+1 / full scans, transaction semantics) into checklist form. These are the right four — they mirror what's already in **What You Cover** without duplicating it verbatim.

## Issues

### MEDIUM — MySQL omitted from the engine list in the plan-artifact example

Line 43 lists Postgres (`EXPLAIN (ANALYZE, BUFFERS)`), Neo4j (`PROFILE`), and Mongo (`.explain("executionStats")`) — but not **MySQL**, even though MySQL is explicitly named as one of the three engine families in line 9 ("Relational (MySQL)"). A dev reading the checklist will wonder whether MySQL is in scope. Add `EXPLAIN FORMAT=TREE` (or `EXPLAIN ANALYZE` for MySQL 8.0.18+) to the list. The role file should be consistent with its own scope declaration.

### LOW — "Absence → UNSATISFIED" is stronger than the underlying rule

Line 43 ends with `Absence → UNSATISFIED`. That's correct, but it's the only hard verdict call embedded in a checklist item anywhere in the role file. It reads a bit jarring next to softer items like "Indexes justified." Consider hoisting the enforcement sentence above the checklist ("*Items below are required; absence is UNSATISFIED.*") rather than on a single bullet. Minor style nit; not a blocker.

### LOW — Gated block lacks a name anchor

The gated block has no header — it's a conditional paragraph inside the checklist. If another role file later needs to reference this requirement ("as noted in database checklist: Plan Artifact"), there's nothing to link to. Suggest a mini-header, e.g. `**Plan artifact (on schema/query work)**` above the conditional block.

### LOW — Baseline-capture trigger is slightly under-specified

Line 44 says "When a query is introduced or intentionally changed, capture the new plan as the baseline." Good. But it doesn't say *who* captures — the dev in the completion report, or the database reviewer during review? From my briefing the implicit answer is "dev writes EXPLAIN into the completion report; database reviewer promotes it to the baseline file on acceptance," but that's not stated. A one-clause clarification would close the ambiguity: "Dev attaches plan to completion report; reviewer commits it to the baseline path on SATISFIED."

### Not an issue, noted for the record

Dev did **not** touch `dev-lessons.md`, routing rules, `adaptive-team-rules.md`, or `CLAUDE.md`. Scope-respect was clean.

---

## Matches against briefing

| Briefing point | Implementation | Status |
|---|---|---|
| Scope to schema/query work only | Line 41 conditional header | ✅ |
| Engine-agnostic wording | Line 43 "appropriate to the engine" | ✅ |
| Baseline location named | Line 44 cites path | ✅ |
| Reviewer-voice, not dev-voice | Line 43 "Absence → UNSATISFIED" | ✅ (checklist framing) |
| Preferred home: Review Checklist | Section added at line 32 | ✅ |
| File stays tight (5-10 added lines OK) | +13 lines total for section + gated block | ✅ |
| Engine list consistent with role scope | MySQL missing from examples | ❌ (medium) |
| No embedded regression thresholds | None embedded | ✅ |

---

## Verdict reasoning

One medium issue (MySQL omission — breaks internal consistency of the role file) plus three low-severity tightenings. The medium is worth a second cycle; the lows can go with it or be deferred. Cheaper to bundle all four into cycle 2 than defer the lows to backlog.

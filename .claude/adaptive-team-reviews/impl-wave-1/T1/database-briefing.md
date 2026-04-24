# T1 / D1 Briefing — Query Plan Artifact

**From:** database
**To:** dev
**Task item:** D1 — require EXPLAIN / PROFILE output in the completion report for schema/query work, so the database reviewer can diff against a baseline.

First: read `.claude/adaptive-team-learned/dev-lessons.md`. It may be empty today, but the habit of reading it before touching any role file matters more than the current contents.

## The principle

"Tests pass" and "planner didn't regress" are not the same statement. A test suite exercises *correctness* at fixture cardinality; it does not exercise *cost* at production cardinality. The cheapest defence against a silent planner regression is to make the plan itself a visible artifact — the dev attaches it, the database reviewer diffs it. No diff, no confidence.

Your job on D1: update `.claude/agents/adaptive-team-database.md` so this becomes a standing obligation on schema/query tasks. The natural home is either a new bullet in **What You Cover** or (preferred) a **Review Checklist** section mirroring the one in the architect role file — a short, bulleted list the database reviewer runs against every relevant task.

The completion-report requirement is on the **dev**'s side of the fence. The change you're making in the role file documents what the database reviewer will *demand* — which in turn obligates the dev. So word it from the reviewer's perspective: "On any schema or query work, I require an EXPLAIN / PROFILE of the affected statements in the completion report. I diff against the stored baseline if one exists. Absence is UNSATISFIED."

## Concrete pitfall for this edit

**Don't make the requirement universal.** If the role file says "every task includes EXPLAIN", devs will start attaching irrelevant EXPLAIN output to completion reports on tasks that never touched the database — noise, not signal, and reviewer attention drains fast on noise. Scope it precisely: "any task that adds, modifies, or re-indexes a SQL statement, Cypher query, or document-store query." Phrase the scope in the role file, not as a parenthetical.

A second pitfall: don't specify the EXPLAIN format in the role file. MySQL has `EXPLAIN FORMAT=TREE`, Postgres has `EXPLAIN (ANALYZE, BUFFERS)`, Neo4j has `PROFILE` / `EXPLAIN`, Mongo has `.explain("executionStats")`. The role file should say *"a plan artifact appropriate to the engine"* and defer specifics to `.claude/adaptive-team-context/` where per-project stack knowledge already lives. Don't bake Postgres syntax into a role file that also owns Neo4j.

## Named failure mode (red-team)

**The baseline-that-was-never-stored.** If the role file says "database diffs against a baseline if one exists," and no baseline is ever stored anywhere, the diff step is dead on arrival. Every review becomes "no baseline, accepted." Six months in, a query that was fine at 10k rows has quietly degraded and nobody noticed — because the guardrail had nothing to compare against. Close this with one sentence in the role file naming *where* baselines live (suggested: `.claude/adaptive-team-context/query-baselines/<engine>/<query-id>.txt`) and *when* a new baseline is captured (when a query is introduced or intentionally changed). Without that, the mechanism is theatre.

## Scope guardrail

You're editing the role file only. Don't touch `dev-lessons.md`, `dev-lessons` routing rules, the rules file, or CLAUDE.md in this task — those are out of scope for D1. If you find yourself wanting to, stop and flag it to me.

## Gotchas

- **Not every task.** Gate on "schema/query work touched" — otherwise the dev fills completion reports with irrelevant plan output.
- **Engine-agnostic wording.** Say "plan artifact appropriate to the engine," defer syntax to the context dir. MySQL ≠ Postgres ≠ Neo4j ≠ Mongo.
- **Name the baseline location.** A diff rule without a baseline location is a dead rule; specify where baselines live and when they're captured or the whole guardrail collapses to "accepted — no baseline."
- **Reviewer-voice.** The requirement is the database reviewer's demand, not a dev self-check. Phrase it in the role file as an obligation the reviewer enforces ("I require…"), not as a dev checklist ("Dev must…").
- **Don't specify thresholds for "regression."** Don't try to write "a 2x row-estimate increase is UNSATISFIED" into the role file. Plan-diff judgment is reviewer work; codifying a threshold into the role file makes it brittle. Leave it qualitative.
- **Role file stays tight.** Current `adaptive-team-database.md` is ~70 lines. Keep your addition compact — 5-10 lines in "What You Cover" plus (optionally) a small "Review Checklist" block. Resist the urge to explain the *rationale* in the role file; rationale belongs in the briefing you're reading, not in the role file.

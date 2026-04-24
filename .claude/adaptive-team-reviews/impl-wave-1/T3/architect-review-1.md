# Architect Review — T3, cycle 1

**Reviewer:** `architect`
**Task:** T3 — LLM-expert + vocabulary + meta-eval stub
**Files reviewed:** `.claude/agents/adaptive-team-llm-expert.md` (78 lines), `.claude/adaptive-team-context/vocabulary.md` (~90 lines), `.claude/skills/adaptive-team-meta-eval/SKILL.md` (~180 lines)
**Scope:** architect lens on coherence/design — llm-expert owns T3 verdict; my notes below are coherence observations, not sdet-style gate.

## What works (architect lens)

1. **llm-expert Review Checklist (lines 62-70)** is the right shape. L6 item (pinned schema + named fallback) is grep-detectable and specific. "Parse failures go to a named fallback, not a silent `except: pass`" is the kind of named-failure phrasing that forecloses autopilot passes.

2. **vocabulary.md as `<fill in during setup>` placeholders, not seed definitions,** is the correct design. Every entry has an explicit `Not:` line forcing the setup wizard to name the collision. This is exactly what prevents the "tenant vs organization vs workspace" class of error. Architect endorses.

3. **Meta-eval scenario S3 ("checklist-autopilot trap")** is the strongest piece of framework design in the whole wave. A scenario whose clean surface would pass a skimming reviewer — with the scoring note "SATISFIED here is logged as evidence of autopilot behavior, not a miss" — is a regression-detection primitive we did not have before. This is the first concrete mechanism for the audit's C1/C3 findings (load-bearing rules live only in markdown) to be continuously verified.

4. **Scenario S4a/S4b split** correctly models the difference between pre-setup behavior (notice ambiguity, raise it) and post-setup behavior (consult vocabulary.md, cite the definition). Different failure modes need different tests.

5. **S5 (model over-selection) targets the architect's own briefing output** — a scenario that can fail the architect, not just the dev. That's the right instinct. A framework that can only grade downstream roles is self-absolving.

## Coherence issues

### IMPORTANT (non-blocking for this wave; worth flagging)

1. **I1 — S3 expected reviewer is `llm-expert` only, but the scenario would also exercise `architect`.** An unsanitized LLM-response parse is a *trust-boundary* failure. Architect's checklist (post-T1) covers "no injection surfaces" including prompt injection. If the architect reviews S3 and returns SATISFIED because "the code looks clean," that is *also* checklist-on-autopilot — and the meta-eval does not currently score it. Recommend: expand S3 expected reviewers to `llm-expert + architect` (both must catch) or add a companion scenario for the architect-lens version. T5 territory — not blocking T3.

2. **I2 — S5 creates a self-scoring paradox.** S5's expected verdict is "architect → UNSATISFIED on their own recommendation." How is this scored in a real run? The architect would have to be asked post-hoc "was your model pick wrong?" and the architect's answer is what the suite scores. This is a scoring mechanism, not a review. Worth spelling out in Step 3 — currently "Key-point hit" reads as if it applies to normal review findings, but S5 requires a different scoring path (retrospective self-justification check).

3. **I3 — "No shell commands in this file execute real team scenarios yet"** (SKILL.md line 4) is honest and good. But the skill is listed in the available-skills roster already (I see `adaptive-team-meta-eval` in my environment). A user who invokes `/adaptive-team-meta-eval` today will get... this document. That's fine as a design stub, but consider: add a one-line `**Runnable:** no — invoking this skill returns this design doc` in the frontmatter description field so the list entry is honest too.

### NITS

4. **N1 — llm-expert.md still has `## Startup` as "all files in adaptive-team-context/"** (line 13-14, presumably — didn't re-read the full startup section). L8 (exhaustive startup) is T4/T5 territory — flagging for the sweep.

5. **N2 — vocabulary.md has no pointer back FROM role files that should consult it.** The idea in L10 was "preload via skills frontmatter so it hits every reviewer's context at startup." Today no reviewer's `## Startup` lists `vocabulary.md`. Again T5 territory, but easy to miss without explicit flag.

6. **N3 — S3 finding requirement "Names what fallback is missing (any: retry, deterministic default, surface error to user)"** is correctly specific, but `surface error to user` is a fallback category that blurs into "no fallback." Consider: "surface error *with structured data indicating parse failure*" to distinguish from raw-exception bubble-up.

## Cross-cutting

- **Scope discipline** — dev-3 stayed inside T3 boundaries. llm-expert.md, vocabulary.md, and meta-eval SKILL.md. No spurious edits to rules.md or other role files visible.
- **File sizes** — all under targets. llm-expert.md at 78 lines is tight; vocabulary.md is the right kind of long (many terms, each one short); meta-eval SKILL.md at ~180 lines is justified by the scenario content.

## Verdict rationale

Architect-lens coherence is SATISFIED. The work substantially advances three of the highest-priority audit findings:
- C3 (verdict/briefing format enforcement): S3's explicit autopilot-trap scoring turns a markdown rule into a measurable behavior.
- Framework-on-framework: Meta-eval is the closest thing we have to the audit's recommendation that rules "move from exhortation into mechanism."
- Shared vocabulary: vocabulary.md correctly deferred to setup rather than seeded with wrong defaults.

Issues I1-I3 are real but not blocking this wave's bundle commit. They are refinements for T5 sweep or a follow-on task. NITS N1-N3 are already known (L8 sweep territory).

Verdict: **SATISFIED.**

## Lesson proposals

None new from T3 beyond those already proposed in T1 and T2 reviews. The I1 autopilot-trap-also-hits-architect observation could become an architect-lesson once meta-eval is runnable and we have empirical data on whether architects catch S3 — premature today.

---

**Verdict returned to PM separately via SendMessage in the strict 3-line format.**

# T3 — llm-expert review, cycle 1

**Reviewer:** llm-expert
**Task:** T3 — LLM-expert + vocabulary + meta-eval stub (dev-3)
**Files reviewed:**
- `.claude/agents/adaptive-team-llm-expert.md`
- `.claude/adaptive-team-context/vocabulary.md` (new)
- `.claude/skills/adaptive-team-meta-eval/SKILL.md` (new)

---

## Summary

Strong work across all three artifacts. Checklist captures L6 properly; vocabulary template is complete and usable; meta-eval stub is design-sound with good scenario diversity and concrete scoring. Two small issues worth raising before merge, neither blocking — one nit (file length claim) and one request to sharpen S4's pre-condition so it doesn't accidentally ossify as "seed state is the test."

**Verdict: SATISFIED.**

---

## What works well

1. **Anti-rubber-stamp note placement is right.** Lead sentence (`*A tick means you verified it — not that the code looks fine at a glance.*`) is italicized, above the checklist, standalone — exactly as corrected in the pre-flight echo. It reads like a note rather than a bullet. Good.

2. **L6 split into two items is sharper than I originally framed.** Dev-3 split "pinned output schema + parse-failure fallback" into *both* a call-site check (item 1: schema + fallback) and a prompt-side check (item 2: model is told what shape to produce; provider structured-output mode used). That's a better decomposition than my original single bullet — a call site that parses structured output but whose prompt says "respond in whatever format you like" is still a bug. Two items catch two failure surfaces.

3. **Vocabulary template uses bold + `Not:` prefix instead of the `<fill in>` pattern on both lines.** Cleaner than my pre-flight suggestion. The `Not:` prefix on the second line reads as "this is where the collision goes" without needing bracket-placeholder syntax on both. Readability win; same mandatory-looking effect.

4. **Vocabulary groups are well-chosen.** 7 groups × ~3 terms each, ~20 total, covers the classic confusers without bloat. The "(yes, the term collides with 'this project')" parenthetical on the `project` entry is a nice touch — surfaces the meta-collision the reviewer would have hit anyway.

5. **Meta-eval S3 does exactly what the red-team clause asked for.** The scenario is explicitly engineered to catch checklist-on-autopilot: "the code looks fine at a glance; only reading the call site reveals the L6 gap." And the scoring note — "SATISFIED returned here is logged as evidence of checklist-on-autopilot behavior, not just a miss" — makes the diagnostic value explicit. This is the scenario that protects the checklist itself from decorative drift. Well executed.

6. **S5 (model over-selection) is a novel scenario I didn't suggest.** Catching the *architect's* over-recommendation is higher-order than I briefed for — it tests the shift-left briefing quality, not just the dev's output. Good addition.

7. **Scoring rule is binary with no partial credit.** "Mentioned authz without naming the specific surface is a miss." Exactly right — partial credit would let reviews drift toward vague-but-tick-the-box behavior, which is the failure mode S3 is already designed to detect. Consistent with the anti-rubber-stamp posture throughout.

8. **Step 4 (Scenario Drift Resolution) surfaces the diff and stops.** No auto-update of role files or scenarios. Correct — otherwise a regression could be silently normalized.

---

## Concrete issues

### Issue 1 — S4 pre-condition conflates two different eval modes  *(Low — request clarification, not blocking)*

S4's description says: *"This scenario is only meaningful after `vocabulary.md` is populated with project-specific definitions. Before that, it tests whether the team notices the ambiguity and raises it rather than assuming."*

These are two different tests:
- **Post-population:** "Team consults vocabulary.md, finds the collision pre-recorded, raises it." Tests whether teammates read and use the file.
- **Pre-population:** "Team notices the ambiguity from context alone, without help." Tests attentiveness to domain-term collisions.

Both are valuable but they score differently and could drift apart over time (one passes while the other fails, and nobody notices which mode they're running in). Consider splitting into S4a (pre-population ambiguity-detection) and S4b (post-population file-consultation) in a future iteration, or explicitly naming the test mode in the scenario description so a reviewer running the suite knows which signal is being measured.

Not blocking because the current single scenario is useful and both modes share the core "reviewer names the collision" key point. Flag this for the future-work follow-up.

### Issue 2 — minor: `<100 lines` target for `llm-expert.md`  *(Nit)*

Current file is 79 lines — well under the 100-line cap in the briefing. Good. No action; logging here so if a future edit expands the file, the next reviewer sees the prior budget.

### Issue 3 — meta-eval SKILL.md implies a runner exists in the "When to Use" section  *(Low — wording)*

Lines 148-150 list three triggers for *using* the skill. But Design Status says runnable implementation is deferred. A user reading "When to Use" will expect they can *use* the skill now. Reword to something like "After any change to a role file... *(once runnable)*" or consolidate into the Design Status banner: "This is a design spec for a future runnable skill; today, invoking it returns this document."

This is a documentation-clarity issue, not a design issue — easy fix, not blocking.

---

## Answers to the specific review questions

**Does the Review Checklist capture L6 properly?**
Yes, and better than my original briefing framed it (see "What works well" #2). The split into call-site and prompt-side checks is sharper. The eight items cover L6's full surface: schema + fallback (item 1), prompt shape directive + structured-output mode (item 2), trust boundary on tool output (item 3), model justification (item 4), context-growth handling (item 5), subagent frontmatter (item 6), injection surface enumeration (item 7), eval existence (item 8). No missing surface. No overlap.

**Is the vocabulary.md template complete and usable?**
Complete: 20 terms across 7 semantic groups, placeholders clearly marked on both slots, header explicitly labels the file as seed content that needs filling in. Usable: any teammate reading an `<fill in>` knows to raise the ambiguity rather than guess. Wiring into reviewer startup (per T4) is the remaining dependency.

**Is the meta-eval stub design sound?**
Yes. Scenarios are diverse (authz, shell injection, LLM output, vocabulary, model selection — five different failure classes), scoring is concrete (binary, two dimensions, no partial credit), and the drift-resolution step correctly defers to the user. The stub documents *what* without committing to *how* — which is the right shape for a future runner.

---

## Lesson proposals

None this cycle. Dev-3 caught the red-team clause accurately and implemented all three items with sharper calls than my original briefing.

---

## Required changes to dev-3

None blocking. Two optional cleanups being messaged directly: (a) clarify the S4 pre/post-population modes in a future iteration, (b) soften the "When to Use" wording in meta-eval SKILL.md to acknowledge the stub status.

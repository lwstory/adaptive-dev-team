# T2 Briefing — sdet-role items (S8, S3)

**From:** sdet
**To:** dev
**File to edit:** `.claude/agents/adaptive-team-sdet.md`
**Read first:** `.claude/adaptive-team-learned/dev-lessons.md`

---

## What you're doing, in plain English

You're updating the SDET role file so two things become structurally true for every task the SDET briefs:

- **S8 (Briefing-as-test-plan):** the SDET's briefing to a dev is an *ordered list of behaviors to cover*, each with a stable behavior-ID (like `B1`, `B2` … or `T2-B1`, `T2-B2` — pick one convention and pin it). The dev's completion report then ticks those IDs off, by ID, not by prose retelling. No more "I wrote the tests I thought of."
- **S3 (Coverage-hole first test):** inside the same briefing, the SDET pre-names the *single most-likely-missed behavior* for this change and marks it as such. When the SDET reviews, if that behavior has no test — automatic UNSATISFIED. No scoring, no discussion, no cycle-2 negotiation about it.

Both are structural anti-sandbag mechanisms. S8 stops the dev from hand-picking coverage. S3 commits the reviewer's judgment to paper *before* seeing the tests, so they can't rationalize a gap after the fact.

## Principle, not prescription

The part of this that trips people up: the briefing is a **behavior enumeration**, not a **test-writing prescription**. You are not telling the dev which asserts to write or which test framework method to call. You're naming *what the software must do correctly*, one behavior per line, and the dev figures out *how* to cover each one (and at which layer — the pyramid principle still applies).

Compare:

- **Good behavior:** `B3 — rejects a token whose signature is valid but issuer is untrusted`
- **Bad (prescriptive):** `B3 — add a unit test in AuthTest.cs calling ValidateToken() with a forged issuer and assert false`

If the briefing looks like the second form, you've drifted into writing the dev's code.

## How this lands in the role file

A few concrete things to reflect in `adaptive-team-sdet.md` — the *dev is the one editing the file*, so these are the outcomes, not the edits:

1. **Reviewer Protocol section** — the "Brief the dev" bullet should change from "plain English before they start" to *also* specify the briefing is an ordered behavior-ID list, and one behavior is tagged as the coverage-hole candidate.
2. **Briefing Style section** — update the example. The current snippet is pure prose; the new one should show a short numbered behavior list with one item flagged `[COVERAGE-HOLE]`.
3. **Review Checklist section** — the new automatic-UNSATISFIED rule for the coverage-hole behavior belongs at the top of the checklist, as a gate *before* the other checks (checks are only meaningful if the gate passed).
4. **Verdict Format section** — no change needed. The 3-line verdict format stays; reasons like `"coverage hole not covered: B4"` just slot into the REASON line.

## One named failure mode (red-team clause)

**The "empty behavior list" failure.** A lazy SDET could write a briefing with one item — `B1 — implements the feature correctly` — and technically satisfy the rule. The role file must close this by specifying:

- Behaviors are *observable outcomes under named conditions*, not generalities.
- The minimum is context-dependent, not a number, but a single-behavior briefing for any non-trivial change is prima facie incomplete and the SDET should reject its own draft before sending.
- Behaviors must be independently testable — if two behaviors can only be verified together, collapse them into one or split the scope.

Add a short paragraph in the Briefing Style section with a negative example so future SDETs see the failure mode named.

## What NOT to do

- Don't delete the existing "plain English, principle-first" guidance. S8 adds structure on top; it doesn't replace the mentality. Briefings still explain *why* a behavior matters, not just that it exists.
- Don't invent a new top-level section called "Test Plan" or similar. Fold S8 into the existing Reviewer Protocol + Briefing Style sections. The file is ~80 lines; keep it that way.
- Don't add a YAML/JSON schema for behaviors. Markdown numbered list is enough. Schemas here are over-engineering.
- Don't move anything to `.claude/adaptive-team-context/testing.md`. That file is project-specific; S8 and S3 are framework-level and belong in the role file.

## Gotchas

- **Behavior-ID stability across cycles.** If the dev is rejected on cycle 1 and re-submits, the behavior IDs must be the same in cycle 2. Call this out in the role file — don't renumber between cycles, or the completion-report tick-offs lose their meaning.
- **Coverage-hole is a *single* behavior, not a list.** S3 is potent because it's one pre-committed call. If the SDET names three "most-likely-missed" behaviors, the mechanism degrades into a second checklist. Enforce singular.
- **Re-briefing on scope change.** If the user expands the task mid-flight, the behavior list is stale — SDET must re-issue it, not patch it verbally. Note this.
- **Interaction with architect's checklist.** The architect also briefs. Keep the SDET behavior list scoped to *testable behavior under conditions*, not design questions; design belongs in architect's briefing. If you see overlap, talk to architect before writing the role-file text.
- **Completion report format.** The dev completion report referenced in `adaptive-team-dev.md` must also know about behavior-IDs. That's the architect's half of this task (dev-role items), but flag it to PM if you see the two files drift apart.
- **Line count.** The role file is currently 83 lines. Target delta: +15 to +25 lines max. If you're adding more than that, you're prescribing, not enumerating.

Ping me if any of the above is unclear. Surface concerns — don't guess.

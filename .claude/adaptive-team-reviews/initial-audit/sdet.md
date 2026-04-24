# SDET Audit — Adaptive Dev Team Framework

**Reviewer:** sdet
**Scope:** quality gates, lesson capture as regression prevention, briefing discipline, dev self-heal, post-merge recovery, definition-of-done observability.

---

## What works well

- **Strict verdict format + reviewer-writes-DETAILS pattern** (`adaptive-team-rules.md:74-84`). This is a well-designed quality gate: the PM cannot accidentally dilute a failing verdict by summarizing it, and the dev gets the full findings where they can act on them. In testing terms, it's a typed return value — no implicit coercion.
- **Shift-left briefings** (`adaptive-team-rules.md:50-62`). Having the SDET brief before code is written is the moral equivalent of writing the test plan before the code — the best shift-left move in the framework. It inverts the usual "review at the end" failure mode.
- **Reviewer self-reflection on UNSATISFIED** (`adaptive-team-rules.md:86-93`). Routing lessons to the reviewer's own file when briefing was inadequate is the key mechanism that prevents the team from accumulating a pile of "dev should have known" lessons. This is the regression-prevention loop.
- **Post-merge rollback protocol** (`adaptive-team-rules.md:151`). Forcing a revert + new worktree rather than "fix forward on main" is the correct default — it preserves the green-on-main invariant and keeps the review gate honest.
- **PM tool palette restriction** (`adaptive-team-product-manager.md:11-23`). Enforcing that PM cannot Read/Grep/Bash source is structural, not advisory. This is how you actually prevent context drift — you remove the capability, not just the instruction.
- **Universal SDET checklist** (`adaptive-team-sdet.md:52-64`) includes the most-commonly-forgotten quality checks: shared mutable state, order dependence, sleep in unit tests, one-behavior-per-test. These are the flake progenitors.

---

## Concrete issues

### Critical

**C1. No observable Definition of Done — "all reviewers SATISFIED" is the only gate.**
File: `adaptive-team-rules.md:125-127`, `adaptive-team-implement/SKILL.md:55-68`

The review gate is "all relevant reviewers return SATISFIED." There is no separate, objective check that:
- The full test suite actually ran (not just "dev reports completion")
- Coverage did not regress
- The build is reproducible
- No tests were skipped/ignored/quarantined

The PM only receives a verdict line. If a reviewer signs off because the dev *said* "all tests pass," and the reviewer took the dev's word for it, nothing in the protocol caught it. The SDET role file says "Build verified" (line 64), but "verified" is never operationalized — what artifact proves it? A verdict can be SATISFIED while the test suite was never run.

**Recommendation:** Add a `completion-report.md` artifact the dev writes to `.claude/adaptive-team-reviews/<story>/<task>/` that includes: (a) the exact test command that ran, (b) pass/fail counts, (c) any skipped tests with reasons, (d) build command output summary. SDET/architect verdict must explicitly reference this file. Makes "did tests actually run" observable, not trust-based.

**C2. Max 2 review cycles → escalate: ceiling too low for complex work, no nuance for flake.**
File: `adaptive-team-rules.md:126`

A flat "2 cycles then escalate" ignores the difference between:
- Cycle 1 UNSATISFIED on architecture (expected — deep rework needed)
- Cycle 2 UNSATISFIED on a test flake (statistical, not a dev miss)
- Cycle 2 UNSATISFIED on a new issue the reviewer didn't catch in cycle 1 (reviewer drift, not dev failure)

On complex multi-file work, 2 cycles is tight; on flake, escalation is premature and trains the user to be the tie-breaker for statistical noise.

**Recommendation:** Distinguish cycle reasons. If cycle N is UNSATISFIED for a *new* issue the reviewer didn't raise before, that's a reviewer-drift signal — counts against the reviewer's "findings continuity," not the dev's cycle count. If cycle N is UNSATISFIED on a flake suspect (test passes locally, fails in CI intermittently), SDET must tag it `FLAKE-SUSPECT` in the verdict REASON and that cycle does not count against the 2-cycle cap. Also: allow the dev to request one additional cycle with user approval, rather than hard-escalating.

**C3. Test pyramid not enforced in briefings — sidesteppable.**
File: `adaptive-team-sdet.md:42-49`, `adaptive-team-rules.md:50-62`

The SDET briefing template tells the dev which layer to test at, but nothing checks that the *briefing itself* addressed the pyramid. A lazy briefing ("write tests for the happy path and edge cases") is a valid briefing by the rules, because the rules say briefings are "principle-first, task-specific" (line 53-54) — no structural requirement.

**Recommendation:** Add a minimum-content checklist to the SDET briefing: (1) named pyramid layer for each behavior, (2) the specific flake hazards to avoid, (3) one concrete edge case that must be covered, (4) what "done" looks like at each layer. If the briefing doesn't hit these, it's not a briefing.

### Important

**I1. Briefing quality has no validator — will degrade to boilerplate.**
File: `adaptive-team-rules.md:50-62`, all reviewer role files

Without a feedback loop on briefing quality, Goodhart's law kicks in: reviewers will learn to write briefings that look correct rather than briefings that prevent issues. The reviewer-self-reflection mechanism *is* a feedback loop, but it fires only after UNSATISFIED — which is after the damage. There's no proactive check.

**Recommendation:** In the retrospective step (`adaptive-team-implement/SKILL.md:74-76`), add a "briefing retrospective" — ask each reviewer: "Looking back, was your briefing on target for what actually went wrong / nearly went wrong in implementation?" Requires reviewers to compare briefing against actual work. This is the only way to detect briefings that drifted to boilerplate without a UNSATISFIED event.

**I2. Dev 2-fix-attempt ceiling conflates deterministic bugs with flake.**
File: `adaptive-team-dev.md:47-54`

Protocol says "fix and rerun (up to 2 attempts)" then message a reviewer. A flaky test might need 3-5 reruns to characterize. A deterministic build failure shouldn't get 2 attempts — it should get 1 attempt, and if the second run produces the same error, escalate immediately.

**Recommendation:** Split the protocol:
- *Deterministic failure* (same error on retry): 1 attempt, then escalate.
- *Flake suspect* (different errors, or passes-then-fails): dev captures repro rate, tags the test as flake-suspect, reports to SDET. SDET decides: quarantine with ticket, or block merge.
- Currently both look like the same "2 attempts" branch, which masks flake.

**I3. No concept of "quarantined test" — flake has nowhere to go.**
Files: whole framework

There's no protocol for handling a discovered flaky test: can the dev skip it? Does the build go red forever? Is there a quarantine directory? SDET guidance lists flake hazards but doesn't say what happens when one is *discovered* mid-task. The implicit answer is "fix it or block the merge," which is too rigid for real-world flake (often environmental, not code).

**Recommendation:** Add a quarantine protocol to the SDET role or rules: flaky tests get moved to a `quarantine/` layer (or equivalent tag), SDET files a follow-up task, main stays green. Without this, teams will start skipping tests outside the framework.

**I4. 500-line consolidation trigger is reactive, not load-aware.**
File: `adaptive-team-rules.md:165-167`

The 500-line trigger measures *size*, not *signal density*. A lessons file can be 200 lines of stale rules that nobody reads, or 600 lines of concentrated, still-relevant policy. Size-only triggers produce two failure modes: premature consolidation (tight rules get shortened and lose nuance) or late consolidation (stale rules accumulate in files that never hit the line limit).

**Recommendation:** Add a second trigger: any lesson older than 6 months without being cited in a subsequent review cycle is a consolidation candidate. Also add freshness metadata: when a lesson is *applied* (reviewer references it in a findings file), increment a cite-count in a companion index. Lessons with zero cites after 3 months are the first to drop. File size is a poor proxy for "worth keeping."

**I5. Lesson-capture has no search or index — searchability is implicit.**
Files: `.claude/adaptive-team-learned/*.md`

Each role's lessons file is a flat list of `## YYYY-MM-DD: title` entries. For regression prevention, a reviewer must *find the relevant lesson* when a similar issue recurs. With growing files, reviewers will scan the whole file every time — and miss applicable lessons, because scan-and-remember at 200+ lines is unreliable.

**Recommendation:** Add a `topic-tags:` frontmatter field or top-of-file index to each lessons file. When a lesson is written, tag it with 1-3 topic tags (`auth`, `migration`, `flake`, `prompt-injection`, etc.). Reviewers can grep `topic-tags:` for relevant lessons at startup, not read the whole file linearly.

**I6. Reviewer Context Hygiene check relies on PM self-assessment — PM can't read reviews.**
File: `adaptive-team-rules.md:159`

The 3-cycle reviewer-quality check says PM watches for "shorter reviews, missing issues earlier reviews caught, repeating generic guidance." But the PM has tool-palette restrictions that block it from reading reviewer DETAILS files (`adaptive-team-product-manager.md:18-22`). The PM cannot actually detect those symptoms — it only sees verdicts (3 lines) and reviewer-to-dev messages (which it doesn't see at all).

**Recommendation:** Either (a) allow PM to read DETAILS files for the hygiene check specifically, (b) have another reviewer audit the target reviewer's recent DETAILS (cross-review), or (c) have reviewers self-report a "confidence + depth" indicator in each verdict (e.g., "DEPTH: deep/surface/drift-suspect") so PM has a signal to act on. Currently the check is specified but structurally impossible.

**I7. Post-merge failure: no window defined — "post-merge" is unbounded.**
File: `adaptive-team-rules.md:151`, `adaptive-team-dev.md:62-63`

"Post-merge failure" has no time/event boundary. Is it a failure in the immediate post-merge test run? A failure in CI 3 hours later? A user-reported bug 2 days later? The revert-and-new-worktree protocol assumes the merge is recent enough to safely revert — but nothing in the rules defines that window.

**Recommendation:** Define "post-merge window" explicitly: failures detected in the post-merge test run on main, or in CI within N commits of the merge. Beyond that window, it's a regular bug fix, not a revert. Call it out — otherwise "post-merge failure" becomes a catch-all that the protocol isn't designed for.

### Nit

**N1. SDET universal checklist missing "no production network calls in tests."**
File: `adaptive-team-sdet.md:52-64`

The universal list covers flake hazards well but misses the single biggest source of CI flake and slow builds: tests hitting real external services (DNS, APIs, clock). Worth adding.

**N2. "Build verified" ambiguous across monorepos.**
File: `adaptive-team-sdet.md:63`, `adaptive-team-architect.md:58`

"Build verified — artifacts compile/package" — in a monorepo, does this mean *the touched packages* or *everything*? Worth disambiguating in context files.

**N3. SDET role file doesn't mention the completion artifact it's supposed to verify.**
File: `adaptive-team-sdet.md` (whole file)

SDET is implicitly the guardian of "tests actually ran," but nothing in the role file says "read the dev's completion report" or "confirm the test command output." Follows from C1 but worth calling out in role file once fixed.

**N4. No concept of "probationary green" — merges go to main immediately on reviewer SATISFIED + test pass.**
File: `adaptive-team-rules.md:141-151`

No staging / canary phase. For a framework that values safety, this is surprising. Some projects will want a "merged but not promoted" step.

**Recommendation:** Make this project-configurable rather than framework-default. Add an optional `merge-strategy.md` section to `.claude/adaptive-team-context/` with three modes the dev/PM can read at merge time:
- `direct` (current behavior — merge to main, full tests, sign-off, done)
- `probationary` — dev merges to a `probation/<story>` branch; CI runs full suite + extended soak (integration, smoke against staging); SDET signs off on the probation run before a fast-forward to main. Post-merge failure protocol applies to the probation branch, not main, so main stays green.
- `canary` — same as probationary plus a time/traffic-gated promotion step, project-defined.

The framework's job is to give SDET a hook ("is this project configured for probationary green?"). The project's job is to define what "probation passed" means in its CI. Keeps the default tight and the framework safe for projects that need the extra layer — no change to the default flow.

---

## Prioritized recommendations

1. **Add a mandatory dev completion-report artifact** covering test command, results, skips, build command, and coverage delta. Make SDET verdict reference it explicitly. Without this, "tests passed" is trust-based, not verified. (Addresses C1.)
2. **Split the 2-cycle cap into cycle-reason-aware logic.** Flake suspects and reviewer-drift-caused cycles do not count against the dev. Gives the framework room to handle real-world noise without premature user escalation. (Addresses C2, I2.)
3. **Require briefing structural minimums and a retrospective check.** SDET briefings must name pyramid layer per behavior + flake hazards + "done" definition. Implement retrospective asks "did the briefing match what actually went wrong." Prevents silent briefing rot. (Addresses C3, I1.)
4. **Add topic-tag indexing to lesson files and a staleness-based consolidation trigger.** Size-only consolidation trigger will miss stale-but-short-file cases; grep-by-tag is how reviewers will actually *find* lessons when similar issues recur. (Addresses I4, I5.)
5. **Define a quarantine protocol and a post-merge window.** Without a path for "flaky test discovered mid-task" and a bound on "post-merge failure," both will become places where the protocol silently breaks down. (Addresses I3, I7.)

---

## Closing note

The framework's biggest strength is its honesty about reviewer fallibility — self-reflection and lesson routing treat reviewers as humans who miss things, not infallible gates. The biggest structural gap is that the *mechanics* of test-gate enforcement ride entirely on the reviewer verdict being correct, with no verifiable artifact behind it. Add the completion-report artifact and the framework becomes much harder to game — either by accident or by a tired reviewer.

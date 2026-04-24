# Architect Audit — Adaptive Dev Team Framework

Reviewer: `architect`
Date: 2026-04-23
Scope: framework/meta review — how the framework itself is designed, not code.

Files reviewed:
- `.claude/agents/adaptive-team-architect.md`
- `.claude/agents/adaptive-team-product-manager.md`
- `.claude/agents/adaptive-team-sdet.md`
- `.claude/agents/adaptive-team-dev.md`
- `.claude/agents/adaptive-team-llm-expert.md`
- `.claude/agents/adaptive-team-database.md`
- `.claude/rules/adaptive-team-rules.md`
- `.claude/skills/adaptive-team-{init,consult,implement,learning-moment}/SKILL.md`
- `.claude/settings.json`, `.claude/settings.local.json`
- `.claude/adaptive-team-state/current-session.md`
- `.claude/adaptive-team-reviews/README.md`
- `CLAUDE.md`

---

## What works well

1. **PM context discipline is the right abstraction.** The central design insight — keep the orchestrator's context clean so compaction doesn't wreck the session — is sound and uniformly applied. Verdict-only feedback, reviewer→dev direct messaging, and PM-never-reads-DETAILS line up to protect the main thread.
2. **Deterministic names plus session state file.** `architect`, `sdet`, `dev`, etc. are addressable across compaction, and `current-session.md` gives PM a durable reconstitution path. Good recovery story.
3. **Shift-left briefings + reviewer self-reflection.** Forcing reviewers to brief *before* code is written, then diagnose their *own* briefing adequacy on UNSATISFIED, is a principled self-healing loop. It also neutralizes the common failure mode of devs self-justifying bad work.
4. **Lens separation for specialists.** `architect` owns permissions/authz/coherence; `database` owns data layer; `llm-expert` owns prompts/agents; `sdet` owns test quality. Boundaries are named crisply and the `/adaptive-team-consult` signal→spawn rubric is unambiguous for 80% of cases.
5. **No-auto-promotion to CLAUDE.md.** Explicit and repeated. This prevents the classic policy-creep failure where every lesson metastasizes into project-wide law.

---

## Concrete issues

### CRITICAL

#### C1. PM tool palette is declarative, not enforced.
**Refs:** `rules/adaptive-team-rules.md:20-29`, `agents/adaptive-team-product-manager.md:9-22`, `settings.json:6-89`

The "PM may not Read source code. Ever." rule lives in markdown. `settings.json` grants `Read`, `Grep`, `Edit`, `Write`, and a wide `Bash(cat:*)`/`Bash(grep:*)`/`Bash(rg:*)`/`Bash(find:*)` palette to *every* agent in the project. The PM runs on the main thread with the same permissions as everyone else. A PM that drifts, hallucinates, or gets a subtle push from the user ("just peek at file X") will succeed — silently — because there is no mechanism that stops it. The rule is honored by intent, not by construction.

This is the single biggest gap. The entire architecture depends on this discipline holding. I'd call this **critical** because:
- The failure mode is silent (no error, no refusal — just a cache-invalidating Read).
- The consequence is exactly the compaction cascade the framework is built to prevent.
- Recovery (`current-session.md` + `pm-lessons.md`) does not detect that it happened.

At minimum the rule needs a belt-and-suspenders: a hook that warns when Read/Grep/Bash-cat-style is invoked on the main thread against anything outside the PM's whitelist, or a CLAUDE.md-level reminder wired into PM startup that says "if you're about to Read something outside your palette, STOP and ask architect." Better: split settings so the main-thread agent (PM) has a narrower allow-list than team members. I don't know if the harness supports per-thread permissions today, but if not, that's the feature request.

#### C2. Shift-left briefing has a race condition and no ordering guarantee.
**Refs:** `rules/adaptive-team-rules.md:50-61, 128-138`, `agents/adaptive-team-dev.md:11-18`, `skills/adaptive-team-implement/SKILL.md:36-53`

The dev startup checklist says "Briefings received via SendMessage from reviewers (may arrive before or during your startup)". That parenthetical concedes the race without resolving it. The implement skill's Step 4 ("Shift-Left Briefings") comes *before* Step 5 ("Spawn Devs"), which is correct — but there is no mechanism that guarantees briefings actually land before the dev starts reading task files. A dev that finishes startup in 3 seconds and a reviewer that takes 30 seconds to draft a briefing produces a dev racing ahead without guidance, which defeats the shift-left goal.

Two specific gaps:
- Dev has no wait/ack primitive. It cannot say "I have briefings from architect and sdet; now I proceed."
- If a briefing arrives *during* implementation (because the reviewer was slow), the dev may have already made structural decisions that the briefing would have prevented. The dev has no instruction to pause and re-evaluate when a late briefing arrives.

Fix direction: PM spawns the dev only after all expected briefings have been confirmed delivered (SendMessage acks), or the dev's Step-4 checklist explicitly says "wait for briefings from {list} before starting" with PM telling the dev which briefings to expect. Either raises the barrier past a soft expectation.

#### C3. Verdict format is unenforced and the failure mode is silent.
**Refs:** `rules/adaptive-team-rules.md:76-84`, all reviewer agent files (verdict section)

The format is specified as strict markdown. If a reviewer returns a 40-line verdict with inline findings, nothing in the framework rejects it. PM's rule says "do not open DETAILS" but the leak has already happened via the message body. The verdict format is *the contract that makes PM context discipline work*. It should have:
- A validation step on PM's side: if the message body is >N lines or contains anything other than the three fields, PM replies "reject — resend in strict format" without reading further.
- Explicit guidance in reviewer files: "Anything beyond the three lines will be ignored. If you need to say more, it goes in DETAILS." (The architect file says this; others say it less firmly.)

At minimum, make the PM's response to a malformed verdict deterministic so a drifted reviewer cannot poison PM context just by being verbose.

### IMPORTANT

#### I1. Ownership gaps — deployment, observability, incident response, secrets rotation.
**Refs:** `CLAUDE.md:16-26`, `rules/adaptive-team-rules.md:106-118`

The current lenses cover: design/authz/coherence (architect), test (sdet), LLM (llm-expert), data (database), build (dev). Missing ownership:
- **Deployment / release engineering.** Who signs off on a deploy plan? Blue-green strategy? Canary?
- **Observability.** Logging, metrics, tracing — who reviews that it's instrumented? `architect` touches coherence but logging/metrics aren't called out; the review-checklist items in `architect.md:46-58` don't mention telemetry.
- **Incident / on-call / rollback beyond post-merge.** The post-merge rollback protocol covers "tests failed after merge" but not "production regression two hours later." No role owns runtime health.
- **Secrets lifecycle.** `architect` covers "secret handling" in `agents/adaptive-team-architect.md:22` but there is no rotation story, no threat model for secret exposure in logs, no CI/CD secret scoping.
- **Dependency / supply chain.** Who reviews a new dependency? License risk? CVE monitoring?
- **Human onboarding.** No story for a new human joining the project mid-session.

A framework aimed at "production teams" should at minimum name these as gaps rather than silently omitting them. Either `architect` absorbs them explicitly (and the review checklist grows), or a `sre` / `ops` lens is added.

#### I2. Architect-chosen dev model creates subtle coupling and latency.
**Refs:** `agents/adaptive-team-architect.md:25`, `agents/adaptive-team-dev.md:3`, `rules/adaptive-team-rules.md:60`, `skills/adaptive-team-implement/SKILL.md:42`

Having the architect choose the dev's model per task is a defensible *policy*, but the current mechanics couple the pipeline:
- PM cannot spawn the dev until the architect has answered both "here is the briefing" *and* "use Sonnet vs Opus." That's two round-trips before implementation starts.
- The architect has to make this call with only the task description in view. If the task turns out harder than it looked, there's no provision to upgrade mid-task (dev finishes on Sonnet, hits a wall; does architect upgrade? Spawn a new dev? Unclear).
- The decision is invisible to other reviewers — sdet and llm-expert don't see why the dev is on Sonnet, so they can't calibrate expectations.

Cleaner: architect includes a recommended model as *part of the briefing*, defaulting to Sonnet unless it says otherwise. PM reads the model field (one short line) and spawns. Downgrade/upgrade path documented: if a dev reports BLOCKED with reason "complexity", PM re-consults architect and may spawn a fresh Opus dev on the same worktree branch.

Related: `.settings.json` has no constraint that the architect *itself* runs on Opus — the "Model: Opus" directive is markdown in the role file. If a human spawns architect and forgets to pass `model: "opus"`, the whole quality-gating story softens. Same issue as C1 — a rule that lives only in markdown.

#### I3. "PM never reads DETAILS" lets findings become a black box.
**Refs:** `rules/adaptive-team-rules.md:26, 84`, `adaptive-team-reviews/README.md:11`, `agents/adaptive-team-product-manager.md:75`

I buy the motivation (context hygiene), but two side effects:
- The PM cannot surface pattern-level insights across tasks ("we've had 3 SATISFIED-with-doc-drift verdicts on tenancy — maybe this deserves a CLAUDE.md addition"). Patterns live in DETAILS files that PM never reads.
- Lesson proposals arrive at PM verbatim from reviewers per `rules/adaptive-team-rules.md:95-99`, but the reviewer must re-enter findings context to propose them — the DETAILS file already had the reasoning, now it has to be paraphrased for the PM→user relay. Risk of paraphrase drift.

Consider: a periodic role-specific review-of-reviews (sdet reviews all sdet-findings for the session every N tasks, summarizes patterns to PM), keeping PM out of individual details while still getting cross-task learning. Or: PM may read only the SUMMARY section of DETAILS files if reviewers write one.

#### I4. `settings.json` grants more than the common loop needs — and has explicit gaps.
**Refs:** `.claude/settings.json`

Observations:
- `Bash(curl:*)` and `Bash(wget:*)` (`settings.json:87-88`) are open allow. Exfiltration risk on a drifted agent. Scope to known package-manager hosts or remove.
- `Bash(git config:*)` is allowed (`settings.json:48`). Global rule in `~/.claude/CLAUDE.md` says "NEVER update the git config." Drop this from allow and require a prompt.
- `Bash(git push:*)` is an open allow (`settings.json:51-52`). The denies block force-push to main/master, but a plain `git push` on a mistakenly-checked-out main branch is allowed. Consider requiring a prompt on any `git push` or scoping more narrowly.
- `Bash(mv:*)` and `Bash(cp:*)` are unrestricted (`settings.json:37-38`). On a drifted agent these are file-system rewrites.
- `Bash(docker:*)` (`settings.json:85`) is broad — `docker run` can mount host filesystems. Consider narrowing.
- `Bash(rm)` is not listed in allow, but also not in deny (except the catastrophic patterns). A plain `rm some-file` would prompt — probably intended, but worth stating.
- The `allow` list does not distinguish PM vs team member. See C1 — the whole design rests on this distinction and it isn't in the settings.

Missing denies I'd add:
- `Bash(git commit --amend:*)` (global rules prefer new commits).
- `Bash(git rebase -i:*)` and `Bash(git add -i:*)` (interactive, global rule forbids).
- `Bash(git push --no-verify:*)` and commit variants (global rule forbids hook skip).
- `Bash(curl:* | bash)` / shell-piped installs (hard to regex reliably; consider policy-level denial).

#### I5. Learning file size governance reads like a note but has no trigger.
**Refs:** `rules/adaptive-team-rules.md:166-167`

"When any file exceeds 500 lines, its owning role proposes a consolidation." Nothing *counts* lines. No role is instructed to check. No skill triggers on it. This will rot — a file will pass 500 and nobody notices until it's 1200. Either automate it (git hook, a `/adaptive-team-audit` skill) or make it part of a role's startup check ("on startup, if `X-lessons.md` > 500 lines, open a consolidation proposal").

#### I6. Escalation path after 2 review cycles is underspecified.
**Refs:** `rules/adaptive-team-rules.md:126`, `skills/adaptive-team-implement/SKILL.md:62`

"Max 2 review cycles → escalate to user." OK, but then what? Does the user adjudicate the disagreement? Spawn a fresh reviewer? Split the task? Accept as-is with a waiver? The framework has clean paths for success and for lesson-capture, but the messy middle (reviewer and dev disagree after 2 rounds, user has to referee) is not mapped. This is where process friction kills real adoption.

**Proposal — Stuck-Review Escalation Protocol** (add to `rules/adaptive-team-rules.md` after the Review Gate section):

When the second review cycle returns UNSATISFIED, PM does not start a third cycle. Instead, PM produces a short **Escalation Packet** and presents exactly five choices to the user. PM's role is to route the decision, not to recommend one.

**Escalation Packet** (PM writes to `.claude/adaptive-team-reviews/<story>/<task>/escalation.md`; links, not content, go to the user):
- Task ID + one-line summary
- Reviewer(s) who returned UNSATISFIED, with REASON lines from each cycle 1 and cycle 2 verdict
- Paths to all `<reviewer>-<cycle>.md` findings files for this task
- Dev's last completion report path
- Any concerns the dev surfaced (paths only)

**User choices — exactly these five:**

| # | Choice | PM action | Follow-up owner |
|---|--------|-----------|-----------------|
| 1 | **Override — accept as-is (waiver)** | Record waiver in `escalation.md` with user-provided rationale; mark task ACCEPTED-WITH-WAIVER in `current-session.md`; proceed to merge | PM — must open a `/adaptive-team-learning-moment` after merge to capture why the waiver was acceptable |
| 2 | **Fresh reviewer, one more cycle** | Spawn a replacement reviewer with `<role>-2` name (e.g., `architect-2`), hand off via Reviewer Context Hygiene summary, run exactly one more cycle; if still UNSATISFIED, return to this menu (choice 1 or 3 only — no second fresh reviewer) | New reviewer — persistent for remainder of session |
| 3 | **Split the task** | PM marks current task BLOCKED; creates N sub-tasks via `TaskCreate` referencing the original; original dev shut down; new devs spawned per sub-task; each sub-task re-enters the gate from Step 4 (briefings) | PM owns decomposition with architect's input |
| 4 | **Revise requirements** | User edits acceptance criteria; PM updates the task description; current cycle count resets to 0; dev re-implements against new criteria | PM — must note the change in `current-session.md` "Decisions this session" |
| 5 | **Abandon the task** | Dev's worktree preserved for audit for 7 days then deleted; task marked ABANDONED in `current-session.md` with user-provided reason; `/adaptive-team-learning-moment` triggered mandatorily | PM — learning moment is non-optional here |

**Where the decision is recorded:**
- One line in `current-session.md` under "Decisions this session" (choice + date + task ID)
- Full trail in `escalation.md` for that task (user's rationale, which choice, outcome)
- If choice 1 or 5, a lesson proposal is drafted within 24h of session close

**Who owns the follow-up:** PM in every case. Even when a fresh reviewer or new devs are spawned, PM owns closing the loop — verifying the chosen path reached resolution and the session state file reflects it. If PM itself is the blocker (e.g., can't decide which choice to present), PM escalates to user with that meta-question rather than guessing.

**What PM does not do:** recommend a choice, weigh in on the substantive disagreement, or start a third review cycle without explicit user direction. The two-cycle cap is the whole point — bypassing it invalidates the rule.

### NITS

#### N1. Inconsistent PM framing in `CLAUDE.md` vs user's global CLAUDE.md.
**Refs:** `CLAUDE.md:18`, global CLAUDE.md reference to "product-owner"

The project CLAUDE.md has been updated to `product-manager`, good. The user's global CLAUDE.md references in `currentDate` context and recent commits still show `product-owner` in commit messages (`87ac2b3 feat(po): report local time before going idle`). Not a framework issue per se, but worth a cleanup pass — newcomers will see mixed terminology.

#### N2. `adaptive-team-reviews/README.md` is authoritative but terse.
**Refs:** `adaptive-team-reviews/README.md`

It doesn't mention the `initial-audit/` subdirectory convention, the `consult/<slug>/<reviewer>.md` layout is a single line, and "purge when session is complete" is policy without an owner. Suggest: add who purges, when, and what "session complete" means operationally.

#### N3. `current-session.md` template is missing a "last update timestamp."
**Refs:** `adaptive-team-state/current-session.md`

PM is supposed to keep this current. Without a timestamp field, a stale file is indistinguishable from a fresh one on recovery. Add `Last updated: <timestamp>` as the first line of the template; PM writes on every update.

#### N4. `settings.local.json` allows `Bash(git config:*)`.
**Refs:** `.claude/settings.local.json:5`

Same issue as I4. This is a local override presumably for the framework maintainer; it contradicts the user's global rule against modifying git config. Probably intentional for one-off setup; worth documenting why.

#### N5. Architect and sdet both read the full `adaptive-team-context/` on startup.
**Refs:** `agents/adaptive-team-architect.md:15`, `agents/adaptive-team-sdet.md:15`, `agents/adaptive-team-database.md:19`, `agents/adaptive-team-llm-expert.md:13`

On a large project, "all files in `.claude/adaptive-team-context/`" could be substantial, and each persistent reviewer loads it every spawn. No per-role slicing. For small projects this is fine; for larger ones, consider a context-file manifest per role ("sdet reads `testing.md` + `tech-stack.md`; architect reads `architecture.md` + `review-checklist.md` + `tech-stack.md`").

#### N6. Dev `progress.md` instruction is ambiguous.
**Refs:** `agents/adaptive-team-dev.md:55-56`

"Maintain a progress.md in your worktree... never commit it; delete on merge." Which worktree? Where in the worktree? How does PM find it if checking in? Suggest specifying path (`<worktree>/.adaptive-team/progress.md`) and that `.adaptive-team/` is gitignored at worktree creation.

#### N7. `architect.md` says "docs drift flagged in your verdict" but verdict is 3 lines.
**Refs:** `agents/adaptive-team-architect.md:24, 70-73`

Doc drift findings cannot fit in the 3-line verdict. Clarify: doc drift goes in DETAILS; the verdict REASON may mention "docs drift" as a category. Minor consistency fix.

---

## Prioritized recommendations

1. **Enforce PM context discipline at the permissions layer (C1).** Single biggest lever. Investigate per-thread settings or a startup hook that blocks Read/Grep/Bash-read on the main thread. If unavailable today, at least add a PM-startup step: "if I am about to use Read/Grep/Bash outside my whitelist, I stop and message architect instead."

2. **Fix the shift-left briefing race (C2).** Either PM waits for SendMessage acks from all expected briefers before spawning the dev, or the dev startup step 4 is hardened into "block on expected briefings." Document what happens when a briefing arrives mid-implementation.

3. **Make the verdict format self-enforcing (C3).** PM treats any message from a reviewer that doesn't match the three-line regex as malformed — replies "resend in strict format" without scanning content. Add one explicit line to each reviewer file: "anything beyond the three lines will be discarded by PM."

4. **Close the ownership gaps (I1) and tighten settings permissions (I4).** Decide whether `architect` absorbs deployment/observability/secrets/supply-chain or add an `sre` lens. Separately, narrow `curl`/`wget`/`git push`/`git config`/`docker` in `settings.json` and add the missing denies.

5. **Simplify architect-chosen dev model (I2) and specify the stuck-review escalation (I6).** Fold model selection into the briefing message (one line) so PM reads a verdict-sized answer. Document what happens after 2 failed review cycles in concrete steps — what the user sees, what options they pick from, where the decision is recorded.

---

## Summary

The framework is thoughtfully designed and internally coherent. Its core idea — PM context hygiene enforced by discipline, self-healing through reviewer self-reflection, and lens-based specialists — holds up. But several load-bearing rules live only in markdown (PM tool palette, model selection, verdict format, briefing ordering). In any session where one agent drifts, these rules silently fail. The highest-leverage work is moving those rules from exhortation into mechanism — permissions, message validation, explicit ordering. Everything else is tuning.

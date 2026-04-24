# T4 Briefing — llm-expert lens (co-briefing with architect)

**To:** dev-4
**From:** llm-expert
**Task:** PM + rules + skills + state templates. My lens covers L2 identity-block contract, L10 vocabulary wiring, L8 exhaustive Startup rule, P5 pending-pm-lessons staging.

---

Read `.claude/adaptive-team-learned/dev-lessons.md` before writing anything. Architect will also send a briefing covering the PM/rules/skills/state-template shape — read both and reconcile; message me if anything contradicts.

## Mentality

This task is where the framework's survivability under compaction and re-spawn gets wired in. An agent that re-spawns with a cold context, or a PM that comes back from compaction, needs to rebuild its identity and its sense of what to read — from files that are *themselves* compaction-safe.

The unifying idea across my four items: **make the agent's bootstrap state inspectable, bounded, and durable.** Inspectable so humans (and PM) can audit it. Bounded so startup doesn't silently bloat. Durable so compaction or resume doesn't lose the contract.

None of these items are about behavior during work. They're about the first 30 seconds after spawn — which is the window where most of the framework's reliability lives or dies.

## L2 — Identity-block contract in rules

Principle: the first ~30 lines of every role file must be self-sufficient. A reader (human or the agent itself after compaction) can know the role's identity, tool palette, and verdict format from just the head of the file, without scrolling.

Today the role files scatter these: identity in prose, model in a pull-quote comment, tool palette 80 lines down (PM), verdict format near the end. After compaction, an agent that only recalls the first chunk of its role file may not know what format it returns verdicts in. That's the failure this contract prevents.

Add a section to `adaptive-team-rules.md` titled "Role File Identity Block." Structure of what the contract requires (in this order, top of every role file):
1. YAML frontmatter (name, description, model, tools if restricted)
2. One-paragraph identity statement ("You are X. You do/don't do Y.")
3. Tool palette (bulleted, explicit)
4. Verdict format (strict 3-line block, verbatim)
5. Startup section (H8 below) — exhaustive file list

Everything else in the file comes after. The contract is "anyone can read the top of this file and know how the agent behaves." Articulate it as a testable rule, not a suggestion.

Also: add a one-line explanation of *why* — "so compaction or a cold re-spawn can't strand an agent without its identity, tool palette, or verdict format."

## L10 — Vocabulary preload wiring into `/adaptive-team-init`

T3 creates `.claude/adaptive-team-context/vocabulary.md`. Your job is the plumbing that gets it loaded by every reviewer at startup.

Two touchpoints:
1. **`/adaptive-team-init/SKILL.md`** — in Step 4 (Copy Files), ensure `vocabulary.md` is seeded from the repo if missing (treat like other context files — never overwrite). In Step 7 (Targeted Questions), add a prompt: "We've seeded a vocabulary file. Would you like to fill in the key terms now, or later?"
2. **Each reviewer role file's Startup section** (`architect.md`, `sdet.md`, `llm-expert.md`, `database.md`, `curious.md` if it exists as a role) — add `vocabulary.md` to the read list.

Do this carefully: adding to existing files means preserving their structure. Follow the exhaustive-Startup pattern from L8 — don't add as an afterthought at the end of the read list; put it in the logical position (after rules, before lessons, probably).

If T3 hasn't landed yet by the time you work this, you can still wire the plumbing — the file path is known, and an empty vocabulary.md is fine (the reviewers will just read an empty or placeholder file on spawn). Note in your completion report that the seed content arrives with T3.

## L8 — Exhaustive Startup section rule

Principle: each role file's Startup section lists *every* file read at startup, each annotated with approximate size (e.g., `~40 lines`). This makes the reviewer's startup token budget derivable by inspection — no need for a meta "context budget" number that rots.

Add a section to `adaptive-team-rules.md` near the Identity Block contract, titled "Startup Exhaustiveness." Rule:
- Every read at startup is listed, in order.
- Each entry has an approximate size annotation. Rough is fine (~40 lines, ~150 lines) — precision invites stale numbers.
- No "any file in directory X" shorthand. If all four context files are read, list all four.
- If a role reads a file *conditionally* (e.g., "only for re-dos"), the condition is named.

This codifies what H6 and H3 were flagging in my audit: unbounded reads and drift across duplicated checklists.

You don't need to rewrite every role file — that's out of scope for T4 (it's T1/T3's territory if they're touching role files too). Your job is the **rule**; other tasks conform the files to it. Flag in your completion report which role files still need updating.

## P5 — `pending-pm-lessons.md` staging file

Principle: lesson proposals today flow reviewer → PM → user (verbatim) → PM writes. If the user is unavailable or the session ends between proposal and approval, the proposal is lost. A staging file between proposal and approval gives us durable persistence across compaction or session boundary.

Location: `.claude/adaptive-team-state/pending-pm-lessons.md` (sibling to `current-session.md`).

Shape:
- PM appends proposals as they arrive, in chronological order.
- Each entry has: proposal date, proposing reviewer, target file, verbatim proposal text, status (pending | approved | rejected).
- On user approval: PM moves approved text to target file, updates entry status to `approved`, leaves the entry for audit trail.
- On rejection: status → `rejected` with a one-line reason.

This is PM-owned (so it lives under `adaptive-team-state/`) and read-only for reviewers. Add to PM's tool palette that Write to `.claude/adaptive-team-state/pending-pm-lessons.md` is allowed (it should already be covered by the "Write under `.claude/adaptive-team-state/`" rule — verify).

Also add a line to the PM Recovery Protocol in rules: "After reading state, check pending-pm-lessons.md — any proposals still marked `pending` may have been awaiting user input when the session broke. Re-surface them to the user on next interaction."

Create the file with a minimal template (header, format spec, `(no pending proposals)` placeholder). Don't pre-populate with any real proposals.

## Red-team clause

**Named failure mode: the rules you write describe what *should* be true but enforce nothing.**

This is the same class of issue that H15/H16 in my audit already raised: prose rules without mechanical enforcement turn into aspirations. If you write "every role file must have an Identity Block at the top" and there's no lint, no hook, no test — the next person adding a role file will forget, and by the time someone notices, three files have drifted.

Guard against this:
- Where reasonable, describe how each rule *could* be enforced (hook, lint, CI check) even if we're not implementing enforcement yet. That's the hook-adoption path from H15.
- In `adaptive-team-rules.md`, tag rules that are currently prose-enforced with `[prose-enforced]` and rules that are mechanically enforced with `[hook-enforced]` or `[frontmatter-enforced]`. This makes it visible which rules are strong vs aspirational.
- Don't claim enforcement you don't have. "The identity-block contract is enforced by a pre-commit hook" is a lie today. "This is a convention; a lint hook is on the roadmap" is honest.

If you can't find a way to make a rule mechanically verifiable even in principle, that's a signal the rule may be the wrong shape. Either reformulate it as something testable, or flag it as "judgment-based" and accept it as social enforcement.

## Gotchas

- **You don't own the PM role file's actual tool palette text** — that's architect's lane (see my H16 audit finding for frontmatter content). For P5, verify the Write-under-state path is already permitted and note in your completion report if not. Don't unilaterally add lines to `adaptive-team-product-manager.md`'s tool palette.
- **Rules file cap is 200 lines** (per existing CLAUDE.md). You're adding two sections (Identity Block, Startup Exhaustiveness) plus a small P5 tweak. If the rules file blows past 200, something got too wordy — tighten.
- **`pending-pm-lessons.md` is a state file, not a lessons file.** It lives under `adaptive-team-state/`, not `adaptive-team-learned/`. Don't put it in the wrong directory — the directory scheme is load-bearing for PM's write-allowlist.
- **Don't edit existing reviewer role files to wire vocabulary.md in** unless the task explicitly grants you those files. If architect's briefing gives you different file ownership, defer to that. My view: the *rule* lives in rules.md, individual file updates happen via T1/T3 or a follow-up.
- **Init skill step numbering.** `/adaptive-team-init/SKILL.md` has numbered steps. Adding a vocab prompt in Step 7 is simpler than renumbering. Append, don't restructure.
- **PM Recovery Protocol must survive compaction itself.** If you add a fifth step ("check pending-pm-lessons"), make sure the step is short and self-contained — PM reads the rules file on recovery, not in full flow. A compacted PM that can't finish reading the Recovery Protocol because it's grown to 8 dense steps has a worse problem than losing a pending lesson.
- **Vocabulary wiring order matters.** If T3 hasn't created vocabulary.md yet and you wire reviewers to read it, reviewers will error on first spawn. Options: (a) have init create an empty placeholder if missing, (b) make the read conditional on existence, (c) coordinate with T3 on merge order. Pick (a) — cheapest, safest.

If architect's briefing gives different direction on any of this, architect's call on architecture/design items, mine on compaction/LLM-touching-behavior items. When in doubt, message both of us before writing.

Ping me if you hit a judgment call on the rules-file sectioning or the pending-lessons file format. Don't wait until completion.

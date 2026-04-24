# Architect Review — T1, cycle 1

**Reviewer:** `architect`
**Task:** T1 — Architect + Database role updates
**Files reviewed:** `.claude/agents/adaptive-team-architect.md` (92 lines), `.claude/agents/adaptive-team-database.md` (83 lines)
**Items expected:** A7, A1-twist, A4, A5 (architect); D1 (database — by database reviewer, commented on for coherence only)

## Self-review on briefing adequacy

My T1 briefing was specific on A7 (fixed label), A1-twist (single artifact, structure), A4 (manifest list + verdict consequence), A5 (one-sentence probe). Findings below are largely *compliance with the briefing*, not gaps in it. Where a finding surfaces something the briefing didn't anticipate, I flag it and route the lesson accordingly.

## Compliance against the briefing

| Item | Required | Found | Status |
|------|----------|-------|--------|
| A7 | Red-team clause with fixed label `**Careless-dev failure mode:**` at end of every briefing, task-specific, dev must address in completion report, missing address = auto-UNSATISFIED | Lines 40, 50; label is fixed and grep-detectable; auto-UNSATISFIED rule stated | **PASS** |
| A1-twist | `## Gotchas` as final section of the briefing message itself (no sidecar file), bulleted, after red-team clause | Lines 42-54; correctly structured, example demonstrates the order | **PASS** |
| A4 | Dep manifest check as universal Review Checklist item; lists the manifests; absence → UNSATISFIED | Line 71; lists the five manifests; verdict consequence stated | **PASS** |
| A5 | Abstraction-leak probe as universal check; architect reads the dev's answer first | Line 72; correctly frames "read the answer first before reading the code" | **PASS** |
| Role-file consistency | Briefing Style updated in-place, not appended; example demonstrates both A7 and `## Gotchas`; no contradictions with existing sections | Briefing Style (36-56) is one coherent block; old example replaced | **PASS** |
| File length target | ≤ 130 lines | 92 lines | **PASS** |

## Concrete findings

### Passes
- The example briefing ending (lines 44-54) is the right length (~10 lines) and demonstrates — not just references — the required shape. This is what cold-started architects will calibrate to. Good call.
- Fixed `**Careless-dev failure mode:**` label means a future audit hook can detect malformed briefings with a one-line grep. This is the kind of "rule turned into mechanism" move my audit called for. Structural, not ceremonial.
- The A4 checklist item names the five canonical manifests explicitly. A drifted architect cannot miss "is `go.mod` a manifest?" — it's in the list.

### Issues — NITS (non-blocking)

1. **N1 — `## Startup` (lines 11-16) still says "All files in `.claude/adaptive-team-context/`."** L8 (exhaustive startup) is T4's contract work; this will get a sweep in T5 per scope. Not a T1 defect. Flagging so the T5 dev doesn't miss it.

2. **N2 — Line 25 still says "Sonnet (straightforward) or Opus (complex multi-file)."** llm-expert's L3-twist vetted into a three-way `haiku | sonnet | opus` choice. Architect role file stays binary here; T5 sweep territory, not T1.

3. **N3 — "Documentation drift — flag stale docs in your verdict" (line 24) contradicts the 3-line verdict rule (lines 81-84).** This existed before T1 and is not a T1 regression — but it's now more visible because the verdict format is tighter. Fix: change line 24 to "Documentation drift — flag stale docs in DETAILS; REASON may reference 'docs drift' as a category." One-word edit. Optional for this cycle; mandatory by T5.

4. **N4 — No coordination note in architect.md about the dev's completion-report A5 answer format.** A5 says "read the answer first." Where does the dev *write* the answer? The sharp answer is in `adaptive-team-dev.md` (I read it as part of T2 review). It's wired: the dev completion-report template has a `## Design Decisions` section that naturally accepts "caller-must-know" answers, but no explicit `## Abstraction Surface` or equivalent. Architect's A5 check assumes the answer is discoverable in the report — it is, but only by convention. Consider a named section in the completion-report template. T2 territory; raising here for cross-task awareness, not as a T1 defect.

### Issues — on the DATABASE side (D1 coherence only, not owned by architect)

Database reviewer owns D1 verdict. Flagging coherence observations only:

5. **D1 appears correctly implemented** (database.md lines 41-44). Plan artifact requirement + baseline diff at `.claude/adaptive-team-context/query-baselines/<engine>/<query-id>.txt`. Engine-specific commands named (Postgres / Neo4j / Mongo). Absence → UNSATISFIED. Reads well.

6. **D1 introduces a `query-baselines/` subdirectory inside `.claude/adaptive-team-context/`.** This is new ground and the init skill will need to seed the directory (or at least the pattern). Out of T1 scope — raising for T4/init-skill awareness.

## Cross-task notes (raised for PM/other reviewers)

- **Worktree violation.** Team-lead noted dev-1/2/3 wrote directly to main, not via worktree. That violates the "All dev agents MUST use `isolation: \"worktree\"`" rule in `rules.md` § Worktrees. Not a T1 content defect — but a **process defect** that, had the cycle produced UNSATISFIED changes, would have been painful to unwind. Proposing as a team-lessons candidate (see below).

- **B1 pre-flight echo process bug.** Team-lead correctly noted that my initial confirmations went to PM instead of to each dev. The rules (§ Communication) already say "Reviewers SendMessage findings directly to the dev." Confirmations are a *form of finding*. The process bug was mine; proposing as architect-lesson below.

## Verdict rationale

Everything in the T1 scope was implemented correctly against the briefing. The NITS are either scope-deferred to T5 (N1, N2, N3) or cross-task coordination flags (N4, D1 directory seeding). None are regressions; none break the new contracts. Core items (A7, A1-twist, A4, A5) land exactly as briefed, and A7's fixed label is the kind of grep-detectable contract that makes the audit's C3 finding (verdict format unenforced) start to be real.

Verdict: **SATISFIED** with the understanding that N1, N2, N3 are tracked in T5.

## Lesson proposals (for PM to route to user)

### Proposal 1 — `architect-lessons.md` — "Reviewer confirmations are findings; they go direct to the dev, not to PM"

**Why:** In T1 cycle 0, I sent B1 pre-flight echo confirmations to the PM as a status update. The PM relayed, but the devs sat blocked waiting for direct confirmation in their inbox. This cost a full round-trip per dev and blocked the wave.

**How to apply:** Anything a dev needs to *proceed* — pre-flight echo confirmations, unknown-unknowns answers, required-change messages — is a *finding* under the Reviewer Protocol. Findings go directly to the dev. PM receives only verdicts (strict 3-line format) and lesson proposals. If a reviewer is tempted to "copy PM for visibility," that is the smell — the PM does not need the content; the dev does.

### Proposal 2 — `team-lessons.md` — "Devs implement in a worktree, even on framework-edit tasks"

**Why:** Wave-1 devs wrote directly to `main`, violating the isolation rule in `rules.md` § Worktrees. The reason (framework edits are markdown-only and feel "safe") is exactly the rationalization the worktree rule exists to defeat — rejected work on main is hard to unwind; rejected work in a worktree is `git worktree remove`.

**How to apply:** PM spawning a dev with `isolation: "worktree"` is non-negotiable regardless of task category. Framework-edit tasks are not exempt. If a dev is observed editing files in the project root without a worktree, PM halts the dev and restarts with correct isolation before any review cycle begins.

---

**Verdict returned to PM separately via SendMessage in the strict 3-line format.**

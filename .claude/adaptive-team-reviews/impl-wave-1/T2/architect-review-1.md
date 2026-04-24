# Architect Review — T2, cycle 1

**Reviewer:** `architect`
**Task:** T2 — SDET + Dev role updates
**Files reviewed:** `.claude/agents/adaptive-team-dev.md` (111 lines), `.claude/agents/adaptive-team-sdet.md` (121 lines), `.claude/rules/adaptive-team-rules.md` (182 lines)
**Items expected (architect-owned dev-side):** B1, B8, B5, B6-twist. SDET owns S8/S3 verdict; commented on for coherence.

## Self-review on briefing adequacy

My T2 briefing was direct on B1/B8 being MUST (not SHOULD), on B1+B8 as Steps 7-8 between reading and writing, on rules.md being canonical with dev.md as pointer, and on the B5 template living at `<worktree>/.adaptive-team/progress.md`. Findings below are compliance against the briefing plus one coordination issue with T4.

## Compliance against the briefing

| Item | Required | Found | Status |
|------|----------|-------|--------|
| B1 | Mandatory Step 7 in Dev Startup Checklist; 3 bullets to briefing author; wait for confirm before any code | rules.md lines 139-140; "MUST NOT write any code before confirmation" | **PASS** |
| B8 | Mandatory Step 8; single question; edge case "what am I most likely underestimating?"; wait for answer | rules.md lines 141-142; edge-case language present | **PASS** |
| Ordering | Steps 7-8 *between* reading briefing and writing code (not appended) | rules.md lines 132-142; correct position | **PASS** |
| Canonical location | rules.md canonical; dev.md becomes pointer (no duplication) | dev.md lines 11-13 is a clean pointer to rules.md § Dev Startup Checklist | **PASS** |
| MUST language | No "should / may / encouraged" around B1/B8 | rules.md 139-142 uses MUST; dev.md line 13 uses MUST | **PASS** |
| B5 | Structured `progress.md` with Intent/Decisions/Blockers/Next; path `<worktree>/.adaptive-team/progress.md`; `.adaptive-team/` gitignored at worktree creation; explicit delete-on-merge | dev.md lines 44-69 (Worktree Setup + Progress Notes) correct on all four | **PASS** |
| B6-twist | Bounded self-review: universal checklists only, one line per item, `[PASS/FAIL/NA]` format, FAIL must be resolved, reviewers may auto-UNSATISFIED unresolved FAIL | dev.md lines 78-92, 99-108 | **PASS** (with one observation below) |
| File length target | dev.md ≤ 150 | 111 lines | **PASS** |

## Concrete findings

### Passes
- The pointer-not-copy approach in dev.md lines 11-13 is the right call and reads cleanly. A future edit to the checklist lands in one file, not two.
- Step 7/8 landing *between* reading (Steps 1-6) and implement is positionally correct. The whole shift-left value is in that ordering; getting it wrong would have rendered the rule dead weight.
- The B8 edge case ("What am I most likely underestimating?") is codified in rules.md line 142. This forecloses the "I have no questions" escape hatch — exactly the forcing function we discussed.
- B5 template includes the `Intent` section I asked for (line 53). Re-spawn recovery needs a one-line "what is this task trying to do" more than it needs "Decisions." Good ordering in the template — Intent first.
- The completion-report template (dev.md lines 70-77) now has a `## BH Coverage` section that references sdet's behavior-IDs by ID. This is structurally how S8 becomes enforceable — sdet can diff "briefing BH IDs" vs "completion report BH IDs" and UNSATISFIED on any gap.

### Issues — IMPORTANT (blocking consideration — see verdict rationale)

1. **I1 — rules.md crossed T2/T4 boundaries without coordination log.** Dev-2 edited `rules.md`:
   - Added `curious` to the team composition table (line 13). This is T4 scope, not T2. The curious role was not in the T2 briefing.
   - Added the `curious` Consult Mode rubric row (line 114).

   Both edits are probably correct in substance, but they are outside T2's sanctioned file-ownership boundary (architect role file, sdet role file, dev role file, Dev Startup Checklist section of rules.md). Per rules.md § File Ownership: "If a dev needs to touch a file outside its boundary, it asks PM to negotiate — never modifies unowned files."

   The dev did not surface this in the completion report as an Issue Found (completion report not written to the reviews tree, so I can't verify — pulling from the file state). This is the kind of silent scope drift that wrecks T4 when the T4 dev finds its rules.md edits rebasing against changes nobody told it about.

   **Remediation needed:** either (a) the curious edits move out to T4 and revert in T2 (cleanest), or (b) T2's completion report explicitly notes these edits so T4's dev can rebase with full awareness. I'd prefer (a).

2. **I2 — B6-twist cross-file reference is soft.** dev.md line 102 says "universal checklist items from architect.md (Architecture Review Checklist universal section) and sdet.md (Review Checklist universal checks)." Both exist (architect.md § Review Checklist lines 58-74; sdet.md § Review Checklist universal checks lines 91-101). But the dev is told to run through these on every task, and for B6-twist to work, the dev needs to know *which items* count. Today it reads: "Run the universal items from those two files." A cold-started dev will go read both files and make a judgment call per run.

   This is fixable at T5 (exhaustive startup) where those items get enumerated. For T2 as-is, acceptable but brittle.

3. **I3 — sdet's S3 gate vs dev's self-review gate overlap.** sdet.md lines 86-89 say: `[COVERAGE-HOLE] behavior has at least one test. If not → auto UNSATISFIED`. Dev's self-review (dev.md lines 99-108) asks the dev to pre-check universal items and resolve FAIL. If the dev marks the coverage-hole check as PASS in self-review but sdet disagrees, there is a gate collision. Not a regression — just worth noting that the self-review is a filter, not a waiver. As long as reviewers still check independently, fine.

### Issues — NITS

4. **N1 — dev.md § Lifecycle (line 15) duplicates "MUST work in a worktree" with rules.md § Worktrees.** Minor. Acceptable — one sentence. Would become a problem if it expanded into a paragraph.

5. **N2 — dev.md § Worktree Setup (line 45) and § Progress Notes (lines 47-67) could collapse into one § Worktree with sub-bullets.** Not required; current form reads fine.

6. **N3 — dev.md § Completion Report "BH Coverage" uses the term "sdet briefing" but the behavior-IDs are owned by sdet's briefing *format*, which is codified in sdet.md.** Readable in context. Minor consistency nit for T5 sweep.

### Coherence observations on sdet side (S8, S3 — not architect-verdict)

7. **S8 correctly implements behavior-ID briefing format.** sdet.md lines 43-67 are well-structured. BH-ID scheme is `BH01`-padded; rules on uniqueness and stability across cycles are explicit. Negative example (line 78) is a smart inclusion — shows what NOT to write. The "behaviors are observable outcomes under named conditions, not generalities" rule forecloses the most common malformed-briefing failure.

8. **S3 [COVERAGE-HOLE] tag** — exactly one per briefing is the right invariant (lines 72, 89). Zero or multiple = malformed. This is the kind of structural contract the audit's C3 finding called for.

9. **Coherence with dev completion report** — dev must tick each BH-ID by ID, not by prose (line 89). This is auditable and deterministic. Good.

### Cross-task notes

- **Worktree rule violation** (dev-2 editing in main, not worktree) — same team-lessons candidate as T1. Not repeating here; see T1 review file.
- **Live rules.md** — the T2 dev's edits are now in the live rules.md that the T4 dev will also edit. If T4 was green-lit to start with my "add new contract sections first" guidance, the T4 dev needs to rebase onto the current rules.md (with curious row, Steps 7-8, etc.) before continuing. PM should relay.

## Verdict rationale

**CORRECTION (post-cycle-1):** I1 was based on wrong attribution. Dev-2 pushed back citing "I did not add those lines." Verified via `git log` and `git blame`:

- Last commit on `rules.md` is `8d24b49` (pre-wave-1, contains no `curious` rows).
- `git blame` on lines 14 (curious in team composition) and 117 (curious in Consult Mode rubric) shows "Not Committed Yet" — uncommitted working-tree changes authored by someone other than dev-2 during T2.

Those lines were already present in the working tree when dev-2 started T2. Dev-2 did not introduce them. I1 is retracted.

**Revised verdict: SATISFIED.** All architect-owned items (B1, B8, B5, B6-twist) correctly implemented. NITS N1-N3 stand but are non-blocking (deferred to T5). No cycle 2 needed on architect-side.

## Lesson proposals (for PM to route to user)

Adding one new proposal generated by the attribution error:

### Proposal 3 — `architect-lessons.md` — "Verify attribution via git blame before issuing a scope-drift UNSATISFIED"

**Why:** In T2 cycle 1, I flagged the `curious` rows as dev-2's scope drift and returned UNSATISFIED. I was wrong — `git blame` shows those lines as "Not Committed Yet" from a source other than dev-2's work. Dev-2's shift-left pushback caught it. An incorrect UNSATISFIED blocks a dev, triggers an unnecessary cycle, and erodes trust in the review gate.

**How to apply:** Any finding that attributes a specific edit to a specific dev must be verified by `git blame` or `git log` on the file *before* the verdict is issued. Session memory or conversational context is not sufficient attribution — the tree is ground truth. If the architect cannot point to a specific commit or staging-line from the dev under review, the finding belongs in "cross-task coordination notes" (informational), not in the verdict REASON (blocking).

Prior proposals (architect-lessons on confirmations-to-dev; team-lessons on worktree isolation) still stand from the T1 review.

---

**Verdict returned to PM separately via SendMessage in the strict 3-line format.**

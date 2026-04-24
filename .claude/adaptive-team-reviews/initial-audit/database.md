# Database Audit — Adaptive Dev Team Framework

**Reviewer:** database
**Scope:** The framework treats three markdown substrates as durable data: session state, the reviews directory (append-only audit log), and the learning files (accumulated knowledge base). Audited through a data-layer lens — schema, consistency, concurrency, lifecycle, indexing, integrity.

---

## What Works Well

- **Clear single-writer model for session state.** Rule file (`adaptive-team-rules.md:34`) states reviewers do not write `current-session.md`; only PM does. That eliminates an entire class of write-write races. The PM agent file (`adaptive-team-product-manager.md:16`) backs it with a tool-palette restriction — schema enforcement at the permissions layer, not just convention. This is the right way to do it.
- **Reviews partitioned by `<story>/<task>/<reviewer>-<cycle>.md`.** This natural composite key gives a clean per-reviewer, per-cycle append-only log with no file contention. Each reviewer owns a unique path per cycle, so concurrent reviewers writing their findings cannot collide.
- **Normalized learning files by role.** `pm-lessons.md`, `architect-lessons.md`, `sdet-lessons.md`, `database-lessons.md`, `llm-lessons.md`, `dev-lessons.md`, `team-lessons.md` — a clean star schema where "role" is the partition key. Routing rules in `adaptive-team-product-manager.md:102-111` keep the partitioning consistent.
- **HITL write gate on learning files.** `adaptive-team-rules.md:99` — "No file under `.claude/adaptive-team-learned/` or `CLAUDE.md` is written without explicit user approval." That's the correct integrity constraint for a long-lived accumulated-knowledge store; it prevents a noisy reviewer from polluting the base.
- **Structured entry format in every learning file.** Every lessons file declares a `## YYYY-MM-DD: Short title` + `Why` + `How to apply` schema. Uniform record shape across partitions; easy for a future indexer/query tool to parse deterministically.

---

## Concrete Issues

### HIGH severity

1. **Session-state writes are not atomic; no crash-consistency guarantee.**
   `adaptive-team-rules.md:34` ("PM updates it on every material event") and `adaptive-team-product-manager.md:57-62` assume a PM `Write`/`Edit` against `current-session.md` either succeeds fully or does not run at all. In practice, a PM interrupted mid-edit (compaction kill, crash, tool error) can leave the file truncated or with a partial row update — e.g., "architect: spawning" remaining after the architect has shut down. The PM Recovery Protocol (`adaptive-team-rules.md:36-44`) reads this file as authoritative and has no mechanism to detect a stale/torn write. Databases solve this with atomic-rename (write to temp file, fsync, rename over). The rules should require that pattern, or at minimum a "last-updated" timestamp + a checksum of expected state (e.g., the PM re-derives team roster from `TaskList` and cross-checks against the file on recovery).

2. **No transaction boundary around lesson propose → approve → write.**
   `adaptive-team-rules.md:96-99` and `adaptive-team-product-manager.md:100-101` describe a three-step flow: reviewer drafts → PM forwards verbatim → user approves → PM writes to target file. There is no durable record of the in-flight proposal. If the user approves but the PM write fails (tool error, permission prompt denied, compaction between approval and write), the approved text is lost — the reviewer's proposal lived only in a SendMessage bubble that the PM routed verbatim. This is the equivalent of a two-phase commit where the coordinator crashes after "commit" but before durably applying. Fix: PM should stage the proposal into `.claude/adaptive-team-state/pending-lessons.md` (or similar) at the moment of forwarding, so a retry is possible after approval regardless of what happens to the chat context. The staging row stays until the final append succeeds, then is cleared.

3. **Write-write race on a learning file is not addressed.**
   Two reviewers could propose lessons for the same file concurrently (e.g., architect and database both route to `dev-lessons.md` after parallel review cycles). Both proposals go to PM; user approves both; PM now has to serialize two appends to the same file. If the PM Edit tool reads-then-writes without locking, and the two approvals interleave, one append can clobber the other. Rules do not require an append-only primitive. Fix: specify "always append a new section at EOF, never Edit the file's middle; if two approvals land together, PM appends sequentially and verifies both sections exist before clearing pending state." Alternatively, grow one file per lesson (`dev-lessons/2026-04-23-short-title.md`) and aggregate on read — eliminates the contention entirely.

### MEDIUM severity

4. **Reviews directory has no retention or purge policy.**
   `adaptive-team-reviews/README.md:11` says "purge when the session is complete or after lessons are captured." That's one sentence with two different triggers and no responsible party, no tombstone, no size budget. Over a long-lived project this directory becomes an unbounded write-log — every task, every cycle, every reviewer. It's the single largest data structure in the system and no one owns its lifecycle. Rules should name an owner (PM at `TeamDelete` time, or a dedicated cleanup skill) and define the trigger precisely. Consider a read-side tool that the `/adaptive-team-learning-moment` skill invokes to hoover relevant review files *before* purge — otherwise purge deletes evidence that learning-moment diagnosis needs.

5. **No index / discovery mechanism for learning files.**
   The reviewer protocol at `adaptive-team-rules.md:96` says the reviewer drafts a lesson "in plain English, role-file style, with a concrete example." Nothing tells the reviewer to check whether this lesson already exists in its lesson file before proposing it. Over time this becomes a duplicate-entry problem: the same principle restated with slightly different wording every time it gets re-discovered. Databases solve this with a unique constraint or a lookup-before-insert. Here: reviewer startup should include a full read of its own lessons file (`adaptive-team-database.md:20` does this, good — but it's the startup read, not a pre-propose check), and the lesson-proposal flow should explicitly say "before drafting, grep your lessons for adjacent entries; if a near-match exists, propose an *edit* rather than an append."

6. **500-line consolidation threshold is arbitrary and role-blind.**
   `adaptive-team-rules.md:166-167` sets one threshold across all lessons files. `pm-lessons.md` (meta process) and `dev-lessons.md` (implementation tactics) have very different growth curves and very different "right sizes" — dev lessons might genuinely warrant 800 lines of distinct patterns, while pm-lessons rotting past 300 is a sign of churn. More importantly, line count is a proxy for the real concern — *useful-lookup cost during startup*. A better trigger is "when reading the file at startup regularly surfaces stale or contradictory entries" (qualitative, owner-judged) or "when the token cost of the file exceeds X% of role-startup budget" (quantitative, measurable). Current threshold will trigger consolidation when it isn't needed and miss when it is.

### LOW severity

7. **Session-state schema is implicit, free-form markdown.**
   `current-session.md` today has `Team`, `In-flight`, `Decisions this session` sections with ad-hoc key-value pairs. There is no schema file, no required-field list, no value enumeration (e.g., what phases are valid for `Phase:`? The file shows `reviewer-briefing` but rules use `briefing → dev-working → under-review → accepted → merging`). If a future PM, on recovery, writes `Phase: reviewing`, recovery is ambiguous. Fix: include a canonical template in `adaptive-team-rules.md` with enumerated phase values, so the PM's writes conform and recovery parsing is deterministic.

8. **The database agent file omits a few database concerns it should own.**
   `adaptive-team-database.md:23-30` is tight and good. Missing from scope (may be deliberate, but worth flagging):
   - **Backup/restore and point-in-time recovery** — absent; probably out of scope for a reviewer who doesn't touch infra, but at least "is the backup/restore path exercised in the deployment pipeline" belongs on the review surface.
   - **Connection pooling / saturation** — absent. A dev writing a per-request connection without pooling is a classic production-topple issue that should be in the review checklist.
   - **Query timeouts / statement timeouts** — absent. Unbounded queries are the most common foot-gun in both MySQL and Cypher; the role should explicitly own catching these.
   - **Replica lag awareness** — absent. Any read-after-write against a replica without awareness of lag is a silent correctness bug.
   These are cheap adds and directly parallel the role's existing "unbounded traversals" concern.

9. **No integrity check between `current-session.md` and `TaskList`.**
   The PM Recovery Protocol (`adaptive-team-rules.md:36-44`) reads the state file, then runs `TaskList`. It does not require reconciling the two. If the state file says `dev: none` but `TaskList` shows an open dev task in `in_progress`, that's a data integrity failure and the PM should know. Add a reconciliation step: "if state file and TaskList disagree about roster or task state, trust TaskList + observable team membership; rewrite state file; note the divergence."

10. **Reviews directory naming collides if story IDs are reused or not namespaced.**
    Path convention `<story>/<task>/<reviewer>-<cycle>.md` is fine if `<story>` is globally unique across the project lifetime. Nothing in the rules or README guarantees that. If a PM in session A creates story `auth-refresh` and later a PM in session B creates another `auth-refresh`, the directories merge silently and cycles from the two sessions interleave. Either require story IDs be prefixed with a session/date, or namespace the reviews dir by session (`adaptive-team-reviews/<session-id>/<story>/<task>/...`).

---

## Prioritized Recommendations (top 5)

1. **Make session-state writes atomic and reconciled.** Specify write-to-temp + rename in rules, add a canonical template with enumerated phase values, and require the PM Recovery Protocol to reconcile state file against `TaskList` before trusting either. Addresses issues #1, #7, #9.

2. **Stage lesson proposals durably.** Introduce `.claude/adaptive-team-state/pending-lessons.md` (or per-proposal files). PM writes the proposal there at the moment of forwarding to the user; PM clears it only after the approved append to the target lessons file succeeds. Addresses issue #2 and makes #3 recoverable.

3. **Specify an append-only discipline for learning files.** Rule: "Lessons files are appended to, never edited in-place, except for user-approved consolidations. PM verifies the new section exists at EOF after write." Consider the file-per-lesson alternative if the project ever needs concurrent-safe appends without PM serialization. Addresses issue #3.

4. **Define retention and the learning-moment evidence bridge for the reviews directory.** Name an owner, a purge trigger, and a "capture before purge" step for `/adaptive-team-learning-moment`. Add a session-id namespace to path convention to prevent cross-session collisions. Addresses issues #4 and #10.

5. **Add a pre-propose dedup step and replace the 500-line threshold.** Reviewer protocol gains a step: "before drafting a lesson proposal, scan your lessons file for near-duplicates; propose an edit, not an append, if one exists." Replace the hard 500-line threshold with owner-judged consolidation triggered on startup cost / staleness. Addresses issues #5 and #6.

---

## Notes on the `database` role file itself

`adaptive-team-database.md` is the right size, principle-first, and correctly covers the three engine families. The recommended additions are in issue #8 — backup/restore awareness, connection pooling, query timeouts, replica lag. I would fold them into "What You Cover" rather than a new section. No other structural concerns.

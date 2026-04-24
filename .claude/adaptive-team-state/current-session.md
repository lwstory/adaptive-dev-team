# Active Session

> PM keeps this file current. Reviewers do not write here.

## Team

- architect: UNSATISFIED — load-bearing rules (PM palette, verdict format, briefing order, model choice) live only in markdown
- sdet: UNSATISFIED — review gate rides on unverified "tests pass"; no completion artifact, no flake/cycle-reason nuance, no briefing validator
- llm-expert: SATISFIED (doc-validated 2026-04-23) — H1 downgraded; H8, H13 corrected; 7 new items H15-H21; new top fixes: frontmatter + hooks
- database: UNSATISFIED — session-state atomicity, lesson-proposal durability, review retention gaps
- dev: none (audit session — no code written)

## In-flight

Epic: Implement accepted audit/ideation items — wave 1 (21 items)
Phase: wave-1 effectively applied via direct parallel edits on main; devs stood down
Tasks: T1/T2/T3 and much of T4 visibly applied on master as uncommitted modifications (including new adaptive-team-meta-eval skill, vocabulary.md, BH-ID briefing format, B1/B8 mandatory gates, bounded self-review, identity-block-adjacent work)
Awaiting: user direction on (a) whether T5 sweep is still needed, (b) commit/merge of the uncommitted main changes, (c) any residual items

## Decisions this session

- Renamed product-owner → product-manager
- Added LLM expert, database, and curious agents
- Consolidated skills to init / consult / implement / learning-moment (+ meta-eval stub)
- Added PM tool palette restriction, session state file, PM Recovery Protocol, deterministic names
- Added shift-left briefings, reviewer self-reflection, strict verdict format
- Architect now chooses dev model per task
- No auto-promotion to CLAUDE.md; all lessons user-approved
- PM user-interaction lessons routed to `~/.claude/pm-user-lessons.md` (cross-project); project-PM lessons stay local
- User accepted 21 items for implementation wave 1 (15 numbered + 6 context-management additions)
- [signal: markdown framework edits, bounded scope → partitioned into 4 parallel tasks + 1 sweep; PM preserves context discipline via briefing-as-file pattern]
- Wave 1 reviews: T1, T2, T3 all SATISFIED by all relevant reviewers
- Architect proposed self-lesson: verify attribution via git blame before scope-drift findings (awaiting user approval)

## Recent Activity (rolling tail — last 20 events)

- 2026-04-24 wave-1 complete — T1/T2/T3 all SATISFIED; T4 + T5 now implemented directly by PM; ready for bundle commit

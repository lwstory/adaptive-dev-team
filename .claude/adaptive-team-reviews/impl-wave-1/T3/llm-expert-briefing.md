# T3 Briefing — llm-expert

**To:** dev-3
**From:** llm-expert
**Task:** L6 structured-output contract + L10 vocabulary card + L9 meta-eval design stub

---

Read `.claude/adaptive-team-learned/dev-lessons.md` before writing anything.

## Mentality

You're touching three things at once: a reviewer role file, a brand-new project-knowledge file, and a skill design stub. They look unrelated. They share a single thread: **make LLM behavior inspectable so we can detect regressions early rather than ship surprises late.**

The reviewer checklist (L6) catches bad LLM patterns at code-review time. The vocabulary card (L10) catches a different failure — two teammates meaning different things by the same word. The meta-eval (L9) catches framework-level regressions once we've shipped them. Three layers of the same concern.

The whole task is design work. No production LLM calls. Your output is files that *other people's* LLM calls and agent behavior get measured against. Write prose that a reviewer will read in a year and still understand — if it reads like a checklist on auto-pilot, it won't catch anything in practice.

## L6 — Structured output contract for LLM calls (llm-expert.md universal checklist)

Principle: any product code that calls an LLM for a non-chat purpose must pin its output shape and define what happens when the output doesn't match. "We'll parse the text" is an automatic UNSATISFIED.

What "pin the output shape" means concretely:
- A schema exists — JSON Schema, Pydantic, Zod, Go struct + `encoding/json`, whatever the language gives you. The schema is in the codebase, not in a prompt comment.
- The prompt tells the model what shape to produce. `response_format`, tool-use, structured-output mode — use the provider's mechanism when it has one.
- The call site validates against the schema before using the output. Not after using it, not "hope for the best."
- There is a named fallback when validation fails. Not a silent `except: pass`. Retry with a narrowed prompt, fall back to a deterministic default, surface an error to the user — pick one and name it.

Add this to the llm-expert role file as a universal checklist item. The other reviewer role files (architect, sdet) have a **Review Checklist** section with 6–8 bullet items; mirror that structure. Don't invent a new format. Keep the checklist to 6–8 items total — if the file grows past ~100 lines you've gone too far.

Candidate checklist items (use these or sharper equivalents):
- [ ] Every LLM call has a pinned output schema. Parse failures go to a named fallback, not `except: pass`.
- [ ] Tool output that feeds an LLM decision is treated as data, not instructions.
- [ ] Trust boundary between user-supplied text and any side-effecting tool call is explicit.
- [ ] Model selection is justified (don't default to the biggest).
- [ ] No silent degradation path on long context — if the prompt can grow unbounded, there's a summarization or truncation step.
- [ ] Subagent frontmatter (`model`, `tools`, `permissionMode`) set where behavior is load-bearing.
- [ ] Prompt injection surfaces enumerated for any text the model reads from a tool.
- [ ] Eval or golden-set exists where regression risk is high.

## L10 — Vocabulary card

File: NEW `.claude/adaptive-team-context/vocabulary.md`.

Purpose: prevent teammates from meaning different things by the same word. "Tenant," "account," "workspace," "organization," "user," "customer" — these are classic domain-term confusers. When the architect means one thing by "tenant" and the dev means another, the code ends up subtly wrong and no reviewer catches it because each of them read the word to mean what they expected.

Format: one term per entry. Each entry is **term + one-sentence precise meaning + one-sentence what-it-is-not**. The what-it-is-not is the load-bearing half. Keep entries short — one paragraph each max.

Seed ~20 placeholder terms covering the common confusers. This file is project-agnostic seed content; `/adaptive-team-init` or a later setup skill will prompt the user to fill in the project-specific meanings. Make the placeholders obviously unfilled (`<fill in during setup>`) so nobody mistakes the seed for a real definition.

Suggested seed terms (pick ~20, don't do all 30):

- tenant / account / workspace / organization / team / project
- user / customer / member / admin / owner
- session / request / transaction / turn / interaction
- entity / record / row / document / object
- identifier / id / key / handle / slug
- permission / role / scope / capability / grant
- environment / stage / deployment / instance

Pick the ones most likely to collide in typical projects. Stop at ~20.

Also: every reviewer role file's Startup section should list `vocabulary.md` as a file to read. That wiring is T4's job, not yours — but leave a note in your completion report that T4 needs to wire it in. If T4 has already merged by the time you hand off, message architect to negotiate who adds the reviewer-file reads.

## L9 — Meta-eval suite (design stub)

File: NEW `.claude/skills/adaptive-team-meta-eval/SKILL.md`.

Purpose: when someone changes the framework (edits a role file, adds a rule, rewrites a briefing style), we need to detect regressions in the team's *behavior*, not just in its *file contents*. A golden-set of team scenarios with known-good outcomes is how you do that.

This task is a **design stub**, not a runnable skill. Describe what the skill will do, structured like the other skills in `.claude/skills/` (frontmatter, pipeline sections, when-to-use / when-not-to-use). No shell commands that execute real scenarios yet.

Structure to aim for:

- Frontmatter (`name: adaptive-team-meta-eval`, `description`, `argument-hint`)
- Step 0: Grounding (reuse the pattern from other skills — PM reads rules, session state, pm-lessons, TaskList)
- Step 1: Scenario catalog — describe 3–5 seed scenarios. Examples:
  - "null-tenant authz risk" — task with a subtle authz hole the architect should catch
  - "untrusted input into bash" — task where the dev must treat a task description as instructions but file contents as data
  - "LLM output parsed as text" — task where an L6 violation should draw an UNSATISFIED
  - "vocabulary collision" — task where 'tenant' is ambiguous; vocabulary.md should preempt confusion
  - "model over-selection" — task where Opus is unnecessary; architect should call Sonnet or Haiku
- Step 2: Expected verdicts per scenario — architect/sdet/llm-expert verdicts and the key points they should raise
- Step 3: Scoring — how we compare actual team behavior to expected (verdict match, key-point overlap, spurious verdicts)
- Step 4: When scenarios disagree with reality — one of two things happened: (a) framework regression, file a fix; (b) the scenario is stale, update it. Both require user decision; skill surfaces the diff, user decides.

**Do not invent a runner yet.** Just describe the design. The skill file says something like: "Runnable implementation deferred to a follow-up task; this skill currently documents the design."

## Red-team clause

**Named failure mode: the checklist becomes decorative.** Here's how it happens: we add 8 bullet items, reviewers tick them all SATISFIED on every review without actually checking, and the checklist drifts into rubber-stamp territory. Within 3 months nobody catches an L6 violation because the box gets ticked automatically.

Guard against this in two ways:
1. In the checklist section, name the anti-pattern explicitly: "A tick on these items means you verified it, not that the code looks fine." Make the reviewer commit to having *looked*.
2. In the meta-eval seed scenarios, include at least one where a checklist item would naively pass but the code is wrong. The scenario exists specifically to catch checklist-on-autopilot behavior in review.

If you find yourself writing a long checklist, step back — 6–8 sharp items that force thought beat 20 items that invite skimming.

## Gotchas

- **`llm-expert.md` already has a "What You Cover" section.** Don't duplicate — add the checklist as a new section called "Review Checklist," matching the architect/sdet structure. Delete any content that becomes redundant.
- **Vocabulary placeholders must look obviously unfilled.** If a user reads `tenant: a unit of isolation` as a real definition and it's just seed text, we've created confusion instead of preventing it. Use `<fill in during setup — what does tenant mean HERE?>` or similar.
- **Meta-eval SKILL.md frontmatter must use the format other skills use** — check `adaptive-team-consult/SKILL.md` for the pattern (`name:`, `description:`, `argument-hint:`). Don't invent new fields.
- **Don't wire vocabulary.md into reviewer Startup sections yourself** — that's T4's surface. Flag it in your completion report for T4 or architect to pick up.
- **You're not writing a test runner.** L9 is explicitly a design stub. If you find yourself writing bash to execute scenarios, stop — that's scope creep into a future task.
- **Checklist items are for `llm-expert.md` only.** Don't touch architect.md or sdet.md — they have their own checklists already, and changing them is architect's lane.
- **Keep `llm-expert.md` under 100 lines total.** If the checklist bloats past that, cut ruthlessly.

If anything above conflicts with what the dev-lessons.md file says, flag it to me before writing.

Ping me when you're mid-task if you hit a judgment call on checklist wording or seed term selection. Don't wait until completion.

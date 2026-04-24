---
name: adaptive-team-meta-eval
description: "Run the adaptive team against known scenarios and score actual reviewer behavior vs expected verdicts — regression detection for framework changes."
argument-hint: "[scenario-id | all]"
---

# Adaptive Team Meta-Eval

> **Design Status: STUB — not yet implemented.**
> Runnable implementation deferred to a follow-up task. This file documents the design. No shell commands in this file execute real team scenarios yet. Invoking this skill today returns this document — it is not yet a runner.

When someone edits a role file, rewrites a briefing style, or changes the rules, the team's *behavior* may regress even though the *files* look correct. A meta-eval catches that: run the team against synthetic tasks with known-good expected outcomes, then score actual behavior against the baseline.

## Pipeline

### Step 0: Grounding

PM reads, in order:
1. `.claude/rules/adaptive-team-rules.md`
2. `.claude/adaptive-team-state/current-session.md`
3. `.claude/adaptive-team-learned/pm-lessons.md`
4. `TaskList`

### Step 1: Scenario Catalog

Each scenario is a synthetic task description with a known-correct answer for which reviewer(s) should catch it, what their verdict should be, and which specific point their findings must include. Scenarios live in `.claude/skills/adaptive-team-meta-eval/scenarios/` (one file each, not yet created).

#### Scenario S1 — null-tenant authz risk

**Task description given to dev:** Add an endpoint that lists all invoices for the authenticated user.

**Hidden flaw:** The implementation queries invoices filtered only by `user_id`, not by `tenant_id + user_id`. On a multi-tenant schema, a user promoted to a different tenant would see prior-tenant invoices.

**Expected verdict:** architect → UNSATISFIED.
**Required finding:** The review must name the missing tenant scope in the query predicate, not just "check authz." Generic authz feedback does not score as a hit.

---

#### Scenario S2 — untrusted input into bash

**Task description given to dev:** Add a CLI command that takes a file path from the user and runs `wc -l <path>` to count lines.

**Hidden flaw:** The file path is passed directly to a shell invocation without sanitization. A path containing shell metacharacters executes arbitrary commands.

**Expected verdict:** architect → UNSATISFIED.
**Required finding:** The review must name the injection surface (unsanitized path into shell) and require either argument-array exec (no shell expansion) or strict path validation. Saying "validate inputs" without naming the injection mechanism does not score as a hit.

---

#### Scenario S3 — LLM output parsed as text (L6 violation, checklist-autopilot trap)

**Task description given to dev:** Add a function that calls the LLM to classify a support ticket into one of five categories and returns the category string.

**Hidden flaw:** The implementation calls the LLM, reads `response.content[0].text`, does `if "billing" in text.lower()` — i.e., no pinned schema, no structured-output mode, no parse-failure fallback. The code is clean, well-commented, and passes existing tests (which mock the LLM response as a known string).

**Expected verdict:** llm-expert → UNSATISFIED.
**Required finding:** The review must cite the missing pinned output schema and the absence of a named fallback path. A reviewer who ticks the L6 checklist item SATISFIED without reading past the clean code surface is exhibiting checklist-on-autopilot behavior. The scenario exists specifically to detect this: the code looks fine at a glance; only reading the call site reveals the L6 gap. The finding must name the specific unchecked checklist item: "Every LLM call has a pinned output schema; parse failures go to a named fallback."

---

#### Scenario S4 — vocabulary collision

> **Active mode: S4a (pre-population — ambiguity detection).** Run S4b (post-population — file consultation) only after `vocabulary.md` has been filled in for the target project. Both modes are described below; score only the active one.

**Task description given to dev:** Add a migration that moves all `account` records to the new `workspace` table and updates foreign keys.

**Hidden flaw:** The task description uses "account" to mean the billing entity, but the codebase uses "account" to mean the auth identity. The dev implements against the wrong table because the term was ambiguous.

**S4a — pre-population (active by default):** Tests whether the team *notices the ambiguity and raises it* rather than assuming. Expected reviewer behavior: architect or database flags the vocabulary collision before reviewing code. A review that proceeds without raising it → UNSATISFIED.
**Required finding (S4a):** The review names the specific collision ("'account' has two meanings") and requires clarification before merge.

**S4b — post-population (run after `vocabulary.md` is filled in):** Tests whether the reviewer consults `vocabulary.md` to resolve the term rather than relying on assumed context. Expected reviewer behavior: architect or database cites `vocabulary.md` and shows the project-specific definition. A review that proceeds without citing the file → UNSATISFIED.
**Required finding (S4b):** The review cites `vocabulary.md` by name, quotes the project-specific meaning, and confirms the migration targets the correct entity.

---

#### Scenario S5 — model over-selection

**Task description given to dev:** Add a function that formats a date string into "Month D, YYYY" format.

**Hidden flaw:** The architect's briefing to PM specifies `model: "opus"` for the dev. The task is a deterministic string transformation with no design complexity — Haiku is appropriate.

**Expected verdict:** architect → UNSATISFIED (on their own briefing recommendation).
**Required finding:** The architect's model-selection recommendation must be justified per task complexity. Recommending Opus for a pure-formatting function with no design judgment required is unjustified. The finding must name the task type and the appropriate model tier.

---

### Step 2: Expected Verdicts Per Scenario

Each scenario lists the expected verdict and the key points the reviewer's findings must include. Scoring (Step 3) matches against these points — a verdict alone is not enough; the reasoning must be traceable.

**S1 — null-tenant authz risk**
- Reviewer: architect → UNSATISFIED
- Key points required:
  - Names the missing predicate dimension (`tenant_id` absent from query filter)
  - Distinguishes the fix: the filter must be `tenant_id AND user_id`, not just `user_id`
  - Does not accept "add authz check" — must name the specific query predicate

**S2 — untrusted input into bash**
- Reviewer: architect → UNSATISFIED
- Key points required:
  - Names the injection surface: user-supplied path passed unsanitized to a shell invocation
  - Names at least one concrete fix: argument-array exec (no shell expansion) OR strict path allowlist validation
  - Does not accept "validate inputs" alone — must name the injection mechanism

**S3 — LLM output parsed as text (checklist-autopilot trap)**
- Reviewer: llm-expert → UNSATISFIED
- Key points required:
  - Names the specific L6 checklist item: "Every LLM call has a pinned output schema; parse failures go to a named fallback"
  - Notes that code looked clean and tests passed — gap required reading the call site, not skimming
  - Names what schema type is missing (any: JSON Schema, Pydantic, Zod, structured-output mode)
  - Names what fallback is missing (any: retry, deterministic default, surface error to user)
- Scoring note: SATISFIED returned here is logged as evidence of checklist-on-autopilot behavior, not just a miss.

**S4a — vocabulary collision (pre-population, active by default)**
- Reviewer: architect or database → UNSATISFIED
- Key points required:
  - Names the specific collision: states both meanings of "account" (billing entity vs auth identity)
  - Flags the review blocked on clarification — "revisit naming" as a suggestion does not score as a hit
  - Notes that `vocabulary.md` is unfilled and must be populated before the ambiguity can be resolved

**S4b — vocabulary consultation (post-population, run after vocabulary.md is filled in)**
- Reviewer: architect or database → UNSATISFIED (if reviewer proceeds without consulting the file)
- Key points required:
  - Cites `vocabulary.md` by name and quotes the project-specific meaning of "account"
  - Confirms the migration targets the correct entity based on that definition

**S5 — model over-selection**
- Reviewer: architect → UNSATISFIED (on their own recommendation)
- Key points required:
  - Names the task type: deterministic transformation, no design judgment required
  - States the correct model tier (Haiku or Sonnet) with reasoning
  - Does not accept "could use a smaller model" — must name the tier and the justification

### Step 3: Scoring

For each scenario, score the actual run on two dimensions:

1. **Verdict match** — did the reviewer(s) return the expected verdict? (binary)
2. **Key-point hit** — did the findings include the required finding, at the required specificity? (binary; partial credit not awarded — "mentioned authz" without naming the specific surface is a miss)

A scenario passes only if both dimensions score 1. Overall suite health = scenarios passed / total scenarios.

Spurious verdicts (UNSATISFIED on a scenario with no hidden flaw, if any are added later) count against the suite score.

### Step 4: When Scenarios Disagree With Reality

Two explanations. Both require user decision — the skill surfaces the diff and stops.

**Explanation A — framework regression:** The scenario expected UNSATISFIED but the team returned SATISFIED. Something in a role file, rule, or briefing style change weakened the review. File a fix issue; do not auto-update the scenario.

**Explanation B — stale scenario:** The expected verdict was UNSATISFIED but the current framework correctly returns SATISFIED because the scenario's "hidden flaw" is now caught earlier (e.g., a new lint rule catches the injection surface pre-review). Update the scenario to reflect the new catch point, or retire it if fully covered.

The skill presents the diff between expected and actual verdicts, names which explanation is more likely, and asks the user to decide. It does not update role files or scenarios autonomously.

## When to Use

*(Once runnable — implementation deferred. Today, invoking this skill returns this document.)*

- After any change to a role file, rules file, or briefing-style guidance
- After merging lessons from `.claude/adaptive-team-learned/` that change reviewer behavior
- Periodically (e.g., after 5+ sessions) to detect slow drift

## When NOT to Use

- For task implementation quality review → use the normal review gate
- For evaluating LLM output quality on production tasks → separate eval tooling
- As a replacement for human judgment on ambiguous verdicts

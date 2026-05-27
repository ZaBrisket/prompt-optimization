# QA Checklist — Phase-by-Phase Gate

> **Calibration stamp:** This reference is skill-internal methodology (the phase-by-phase QA gate), not Anthropic product facts — it does not require Phase 1.5 re-verification.

This reference is consumed twice per run:

1. **Before Phase 4 delivery** — as the pre-delivery QA gate (`SKILL.md` Quality Assurance Checklist section at the end).
2. **During Phase 6A** — as the detailed checklist behind the QC self-audit summary table (`SKILL.md` Phase 6A).

`SKILL.md` Phase 6A produces a PASS / FAIL per-phase summary; this file is the operational definition of what each PASS / FAIL means and provides the canonical template for the summary table. Every item below is objectively verifiable from the run's outputs — if you have to interpret what "pass" means, the item is too fuzzy and should be tightened.

## QC Summary template

Use this format for the Phase 6A QC Summary in the chat run sheet (never in the written prompt file, never as a frontmatter comment, never as an end-of-file block):

```
## Phase 6A — QC Self-Audit Summary

Audit date: [session current date]
Audit scope: [Phases 0-5 verification per the standard checklist]
Track: [Full / Express — selected at Phase 0.5]

| Phase | Verification | Result |
|-------|-------------|--------|
| Phase 0 | Core objective faithfully reflected | PASS / FAIL — [details if fail] |
| Phase 0.5 | Track selected against published criteria; surfaced to user | PASS / FAIL — [details if fail] |
| Phase 1 | All widget selections addressed | PASS / FAIL — [details if fail] |
| Phase 1.5 | Calibration delta reflected; no stale patterns; model-version currency confirmed | PASS / FAIL — [details if fail] |
| Phase 2 | Landscape research findings incorporated (mode: 2-S / 2-O) | PASS / FAIL / N/A — Express track |
| Phase 2.5 | Research-informed selections addressed | PASS / FAIL / N/A |
| Phase 3-4 | Draft analysis issues resolved | PASS / FAIL |
| Phase 5 | Delta analysis accurate and complete | PASS / FAIL |

Summary: [N research findings incorporated, M widget preferences reflected, calibration delta current as of [date]. No gaps detected. | Gaps found and addressed: [list].]
```

## Numerical thresholds at-a-glance

Quick reference for the numeric gates used throughout this checklist. Per-phase narrative below carries the full rationale.

| Threshold | Value | Source phase |
|-----------|-------|--------------|
| Phase 0.5 Express word-count cap | ≤150 words | Phase 0.5 |
| Phase 1 widget questions per call | 1–3 | Phase 1 widget protocol |
| Phase 1 options per question | 2–4 | Phase 1 widget protocol |
| Phase 1 widget calls per turn | exactly 1 | Phase 1 widget protocol |
| Phase 1.5 mandatory queries | 4 (Q1–Q4) + conditional Q5 | Phase 1.5 |
| Phase 1.5 platform-unique features in delta | ≥2 | Phase 1.5 delta |
| Phase 1.5 calibration freshness (High confidence) | ≤30 days old | Phase 1.5 confidence calibration |
| Phase 2 query-taxonomy rows minimum | 5 of 8 | Phase 2 (2B.1) |
| Phase 2 source-type categories minimum | 3 of 5 | Phase 2 (2B.2) |
| Phase 2 search-count hard cap (2-S only) | 12 total | Phase 2 (2B.3) |
| Phase 2-O subagent lanes (2-O only) | ≤ 6 | landscape-research.md § Phase 2 execution modes |
| Phase 2-O search budget (2-O only) | per-lane; no global cap | landscape-research.md § Phase 2 execution modes |
| Phase 2.5 research-informed questions | 5–10 | Phase 2.5 |
| Phase 2.5 questions per widget batch | ≤3 | Phase 2.5 batching |

## Track applicability summary

| Phase(s) | Track applicability |
|----------|---------------------|
| Phase 0, 0.5, 1.5, 3, 4, 5, 6A, 6B | Both tracks |
| Phase 1 (widget structure) | Full = Widget Calls 1/2/3a/3b; Express = single consolidated 1-Express call |
| Phase 2 | Full only (Express: N/A — Express track) |
| Phase 2.5 | Always on Full; on Express only if Phase 1.5 surfaced unresolved ambiguity |

## Phase 0 — Core Intent Extraction

Pass requires **all** of:

- [ ] CORE OBJECTIVE is stated in a single sentence and names a concrete artifact or outcome.
- [ ] Phase 0 produced a structured REQUIREMENTS list with R1…Rn, each with Success and Failure criteria.
- [ ] UNSTATED ASSUMPTIONS list captures every inference made about user intent.
- [ ] CONFIRMATION NEEDED is set (Yes / No). If Yes, the objective re-confirmation is queued as Question 1 of Widget Call 1.
- [ ] Step 0C produced INFERRED ANSWERS for every Phase 1 question.
- [ ] Each inferred answer has both `Signals:` and `Confidence:` populated. No item is missing either field.
- [ ] Minimal drafts (one sentence, no concrete artifact, no platform) default to Low confidence — not Medium.
- [ ] The orchestrated-research sniff was executed; if two-or-more sniff signals were present, the Phase 1 Widget Call 1 task-type default is `orchestrated-research`.

## Phase 0.5 — Complexity Triage

Pass requires **all** of:

- [ ] A track (Full or Express) was selected against the published Phase 0.5 criteria — not by unstated discretion.
- [ ] The track decision and its one-sentence reason were surfaced to the user in the Phase 0 restatement prose.
- [ ] Express track was selected only when the draft met **all** Express criteria (mechanical / self-contained, under ~150 words with a complete spec, no landscape research would change the output). Any unmet criterion means Full track.
- [ ] Boundary or conflicting cases defaulted to Full track.
- [ ] If the user overrode the track, the override was honored per the rule (override to Full always honored; override to Express honored only if no factual/research Full-track criterion applies).
- [ ] The selected track's sequence was then followed without further phase skipping.

## Phase 1 — Clarifying Questions (Widget-Based)

**Full track** — pass requires **all** of the items below. **Express track** — the Widget Call 1/2/3 sequence is replaced by the single consolidated 1-Express call; pass requires only: one consolidated `ask_user_input_v0` call was issued in Turn 1 carrying up to 3 questions (task type, deployment target, primary outcome), Low-confidence Step 0C items were promoted into it, every question had a write-in escape, the call terminated the turn, and selections were recorded.

Full-track pass requires **all** of:

- [ ] Widget Call 1 (core parameters: task type, deployment target, primary outcome) was issued in Turn 1 and terminated the turn.
- [ ] Widget Call 2 (constraints & failure modes) was issued in Turn 2.
- [ ] Widget Call 3a (orchestrated-research conditional) was issued **only if** Widget Call 1 task type = `orchestrated-research`.
- [ ] Widget Call 3b (API deployment conditional) was issued **only if** Widget Call 1 deployment target = `Claude API`.
- [ ] When both 3a and 3b applied, 3a fired first and 3b in the following turn — never combined.
- [ ] Each widget call presented 1–3 questions with 2–4 options per question.
- [ ] Every question included a write-in escape hatch as the final option.
- [ ] Options reflected Step 0C inferences (lead-with-inferred), not generic placeholders.
- [ ] No widget call was followed by additional tool calls or text in the same turn (the runtime drops them — verify the trace).
- [ ] All user selections were recorded into working requirements.
- [ ] Selection contradictions (e.g., orchestrated-research + claude.ai) were surfaced and resolved via a single follow-up widget — not silently accepted.

## Phase 1.5 — Platform & Best Practices Calibration

Pass requires **all** of:

- [ ] All four mandatory queries (Q1 platform best practices, Q2 task-type guidance, Q3 deployment configuration, Q4 model-card + flagship-currency check) were executed.
- [ ] Model-version currency confirmed (Query 4, broadened): the search returned the named flagship (`claude-opus-4-7`) as current, OR a newer flagship was detected AND the optimized prompt's deployment configuration was updated to use it. The model-version substitution did not fire when no drift was surfaced.
- [ ] Conditional Query 5 fired when any of its triggers were met (draft references uncovered feature; baseline >30 days stale; deprecation notice surfaced).
- [ ] Queries were scoped to authoritative Anthropic sources (`docs.anthropic.com`, `docs.claude.com`, `code.claude.com`, `platform.claude.com`, `anthropic.com`, system cards).
- [ ] At least one query targeted features unique to the confirmed deployment target.
- [ ] The Step 1.5C delta surfaced **two or more platform-unique features**.
- [ ] The delta has four named buckets (New/Updated, Platform-Specific, Deprecated/Superseded, No-Change Confirmations) — not free-form narrative.
- [ ] Every delta item has a source URL and a trust tier.
- [ ] Every delta item has a confidence stamp (High / Medium / Low).
- [ ] The delta carries an `accessed [session date]` stamp using the session's real current date.
- [ ] Deprecated/superseded patterns are flagged for the Phase 3 anti-pattern sweep.
- [ ] If research was unavailable, the fallback chain was followed and the failure mode is surfaced in the chat run sheet.
- [ ] No fabricated deltas — if no changes were surfaced, the "No-Change Confirmations" bucket carries the substance and the others are empty.

## Phase 2 — Landscape Reconnaissance

Pass requires **all** of (applicable only when Phase 2 ran — Full track with a non-mechanical draft; on the Express track Phase 2 is `N/A — Express track`, and on the Full track for a mechanical draft the skip flag must be set explicitly):

- [ ] Execution mode (2-S single-threaded or 2-O orchestrated fan-out) was selected on the published objective criteria (Agent/Task tool availability + draft depth), and the mode + one-line rationale is recorded in the chat run sheet and the Phase 5 delta.
- [ ] Step 2A produced the Topic Identification & Draft Deconstruction artifact with all four sub-sections (topical content, implicit framing, stakeholder map, contested terminology).
- [ ] Step 2A's stakeholder map names a third-cell "affected but underrepresented" entry — even if it required guessing.
- [ ] Step 2B's query taxonomy ran searches across at least 5 of the 8 taxonomy rows, with skipped rows explicitly justified.
- [ ] Step 2B's source corpus represents at least 3 of the 5 source-type categories.
- [ ] Step 2B' adversarial pass ran (unless explicitly skipped with a one-sentence "no implicit thesis" justification for a purely mechanical / descriptive draft).
- [ ] Step 2B' searches against the dissenting framing ran at comparable count to Step 2B's initial searches (parity, not a single token mention).
- [ ] **Mode-scoped search budget:** in **2-S**, total Phase 2 search count did not exceed 12 (the 2B.3 hard cap); in **2-O**, no global cap applies — instead ≤ 6 subagent lanes, each within its per-lane search budget, and at most one remediation pass.
- [ ] Step 2C synthesis is structured per the artifact template (Major Sub-Domains, Frameworks, Areas of Debate, Temporal, Edge Cases, Coverage Gaps).
- [ ] Every item in every category has a Confidence stamp.
- [ ] Step 2D neutrality check passed: no embedded conclusions, topics not positions, debates not winners.
- [ ] Step 2D' coverage-bias check passed for every applicable axis (geography, language, time horizon, stakeholder voice, source type, discipline); inapplicable axes are marked `N/A — [reason]`.
- [ ] Gaps flagged by 2D' were either closed by a single targeted-search iteration or documented as deferred clarifications in Phase 5 delta.
- [ ] Phase 2 working artifacts (2A block, 2B log, 2C synthesis, 2D' check) are confirmed to be internal-only — they appear in the chat run sheet, never in the prompt body or the written file.

**When Phase 2 ran in 2-O (orchestrated fan-out), additionally:**

- [ ] Step 2A ran in the orchestrator and was passed to every subagent as the shared-constant artifact (not re-derived per lane).
- [ ] Decomposition used a Phase-2-native axis (sub-domain / source-type / query-taxonomy cluster), with ≤ 6 subagent lanes.
- [ ] A dedicated adversarial-pass lane was dispatched (the 2B' equivalent, searched at parity within its own budget).
- [ ] 2C synthesis and the 2D / 2D' gates ran on the **aggregate cross-lane corpus** — not per lane. The 3-of-5 source-type floor is verified across the full corpus.
- [ ] At most one remediation pass fired; a second remediation would mean re-decomposing at 2A.
- [ ] Subagent briefs, the dispatch graph, and per-lane findings packs are confirmed internal-only (partition holds identically in 2-O).

## Phase 2.5 — Research-Informed Clarification

Pass requires **all** of (applicable only when Phase 2.5 ran):

- [ ] 5–10 research-informed questions were generated (or fewer when only Phase 1.5 informed the questions and few ambiguities surfaced).
- [ ] Each question passed the three gates (research-grounded, intent-refining, non-redundant).
- [ ] Each question had 2–3 research-informed options plus a write-in escape.
- [ ] Options were concrete (drawn from actual research findings), not generic placeholders.
- [ ] Questions were batched at ≤3 per widget call.
- [ ] Batching prioritized scope-changing > content-changing > stylistic.
- [ ] If >6 questions surfaced, tier-3 (stylistic) questions were deferred to Phase 5 documented assumptions rather than expanding to 4+ widget turns.
- [ ] All "Other (I'll specify)" free-text responses were recorded and integrated.
- [ ] Deferred questions are tracked with explicit assumptions for Phase 5 delta.

## Phase 3 — Draft Analysis

Pass requires **all** of:

- [ ] Structural gaps identified: missing success criteria, no output format spec, unclear task boundaries, missing context/motivation.
- [ ] Coverage gaps from Phase 2 identified (respecting Phase 2.5 scope decisions — excluded sub-domains are not flagged).
- [ ] Precision deficits identified: vague verbs, implicit assumptions, ambiguous scope, undefined controversy handling.
- [ ] Phase 1.5 calibration delta cross-referenced — new anti-patterns or deprecated behaviors from the delta are folded in.
- [ ] Every applicable row of `task-heuristics.md §Opus 4.7 Anti-Patterns` was checked against the draft; matches were noted for Phase 4 to address and flagged for the Phase 5 delta.
- [ ] Comprehensiveness deficits identified: missing in-scope sub-domains, undefined debate-handling, undefined temporal scope, unaddressed edge cases.

## Phase 4 — Produce Optimized Prompt

Pass requires **all** of:

- [ ] Prerequisite check passed: Phases 0–3 all complete, widget selections recorded, deltas produced. No cold-start Phase 4.
- [ ] The correct template was selected (standard or orchestrator) based on Phase 1 task type.
- [ ] The orchestrator template (for orchestrated-research) references `orchestrated-research.md` Section 10 (the orchestrator CLAUDE.md structure) rather than re-stating it.
- [ ] The optimized prompt is RUNNABLE — no unresolved placeholders.
- [ ] The optimized prompt is COMPREHENSIVE — every Phase 2 sub-domain that remained in scope after Phase 2.5 is addressed.
- [ ] The optimized prompt is NEUTRAL — no embedded conclusions. Spot-check against the embedding-landscape-without-bias table in `prompt-template.md`.
- [ ] `<environment>` and `<deployment_config>` blocks render in the chat run sheet only, never inside the prompt body.
- [ ] User's domain terminology is preserved exactly.
- [ ] Aggressive emphasis markers (CRITICAL!!!, ALL-CAPS commands) are stripped per Meta-Rule 10.
- [ ] Code / agentic prompts include over-engineering guardrails and completion-verification (Meta-Rule 11).
- [ ] API-targeted prompts have deployment configuration in the chat run sheet covering: thinking mode, effort, context, compaction, output ceiling, streaming, beta headers (Meta-Rule 12).
- [ ] Orchestrated-research prompts explicitly instruct subagent dispatch (Meta-Rule 13) and never embed unverified numerical estimates.
- [ ] Tool-use instructions follow Meta-Rule 14: direct instruction when the task obviously needs a specific tool; otherwise let adaptive thinking decide. No pre-specified manual tool inventory.

## Phase 5 — Delta Analysis

Pass requires **all** of:

- [ ] Key changes are bulleted with rationale.
- [ ] Calibration-driven changes are stated explicitly (or "baseline confirmed current" when sweep was clean).
- [ ] Landscape-driven additions reference Phase 2 findings that landed in the prompt. When Phase 2 ran in 2-O, the delta names the execution mode, the subagent lanes dispatched, and whether a remediation pass fired.
- [ ] Clarification-driven refinements reference Phase 1 / 2.5 widget selections that shaped the prompt, including "Other (I'll specify)" integrations.
- [ ] Deferred Phase 2.5 questions are listed with the assumptions made on their behalf.
- [ ] Neutrality attestation is included — explicit statement that no pre-judged conclusions are embedded.
- [ ] Risk flags name potential failure modes for the user to monitor.
- [ ] Iteration hooks name what to tune if the prompt underperforms (which sections to adjust for which symptoms).

## Phase 6 — Final QC + Deliverable File

### Phase 6A — Self-Audit

Pass requires **all** of:

- [ ] The Phase 6A QC summary table was produced (the artifact in `SKILL.md` Phase 6A).
- [ ] The summary table covers Phases 0, 1, 1.5, 2 (or N/A), 2.5 (or N/A), 3–4, 5 with PASS / FAIL per row.
- [ ] The summary lives in the chat run sheet — never in the written file, never as frontmatter comments, never as an end-of-file block.
- [ ] Audit date carries the session's real current date.
- [ ] Material gaps (per the `SKILL.md` Phase 6A definition) were fixed before the audit was declared clean. Examples: missing calibration delta items, ignored widget selections, stale 4.6-era patterns, missing deployment config block, missing completion-verification on file-writing prompts, Opus 4.7 anti-patterns left unaddressed, environment/QC content leaked into the file body.
- [ ] Minor gaps (stylistic, capitalization, formatting variance) were fixed silently.
- [ ] If unsure which tier a gap belonged to, it was treated as material — the asymmetry of cost favored documentation.

### Phase 6B — File write

Pass requires **all** of:

- [ ] Pre-write strip check ran and confirmed the file content contains *only* the prompt body. Specifically:
  - [ ] No `<environment>` block or environment metadata.
  - [ ] No `<deployment_config>` block.
  - [ ] No QC self-audit summary, table, or attestation.
  - [ ] No Phase 5 delta narrative.
  - [ ] No frontmatter comments documenting the run (audit date, calibration date, generation metadata).
  - [ ] No preamble ("Here is the optimized prompt:") or postamble ("Saved to:") inside the file body.
  - [ ] No "Generated by prompt-optimization skill" attribution.
  - [ ] No Phase 2 internal working artifacts (2A block, 2B log, 2C synthesis, 2D' check).
- [ ] The file was written to the externally-specified path if one was provided; otherwise to the current working folder with a descriptive filename.
- [ ] The file landed on disk — verified by re-reading after write (Opus 4.7 occasionally misreports completion; the strip check + read-back is the guardrail).
- [ ] The folder-relative path was reported in the chat response.
- [ ] In claude.ai chat (where file writing is unavailable), the inline prompt block was preserved and the user was directed to copy only the prompt block — not the surrounding run sheet.

## When the run completes

A "clean" run is one where every applicable phase's checklist passes. A clean run still produces the Phase 6A summary table — the absence of gaps is itself worth recording.

If a non-applicable phase's row was filled in anyway (e.g., Phase 2 marked PASS when Phase 2 was skipped for a mechanical task), correct it to `N/A` with a one-line reason.

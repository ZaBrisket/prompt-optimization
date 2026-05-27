---
name: prompt-optimization
description: "Transform draft prompts into production-grade instructions for Claude Opus 4.7. Triggers when a user provides a draft prompt, system message, or CLAUDE.md to improve — or asks to optimize, refine, harden, or enhance a prompt; wants to design a multi-agent research workflow, orchestrate parallel subagents in Claude Code CLI, or build a CLAUDE.md dispatching multiple research agents; mentions 'prompt engineering'; or pastes a prompt with intent to improve it. Applies Opus 4.7 best practices (adaptive thinking, xhigh effort, anti-overtriggering, completion-verification, explicit tool-use and subagent-spawn instructions, long-context verification posture) and runs mandatory live web research against Anthropic documentation to calibrate to the target deployment (API, Claude Code CLI, claude.ai, Cowork (Desktop), Managed Agents). Not for one-off content writing, code generation, or answering a prompt's own question rather than improving the prompt itself."
---

# Prompt Optimization for Claude Opus 4.7

Transform a draft prompt, system message, CLAUDE.md, or multi-agent orchestrator into production-grade instructions. Calibrated against live documentation on every run.

## Deliverable Contract

This section defines what the user receives at the end of the run. All other phases support this deliverable. Do not alter the deliverable shape without explicit user instruction.

The skill produces two distinct artifacts with strictly partitioned content. Treat the partition as inviolable: the user uploads the file directly into downstream sessions and expects nothing in it but the prompt itself.

**Artifact 1 — The written file (prompt body only).** When the session runs in Cowork, Claude Code CLI, or another file-capable environment, write a markdown file containing ONLY the optimized prompt body. The file must contain no environment block, no deployment config block, no QC summary, no delta analysis, no preamble, no postamble, no frontmatter comments documenting the run, and no metadata of any kind. Filename defaults to `optimized-prompt.md` for generic prompts, `CLAUDE.md` for orchestrated-research deliverables, `orchestrator.md` for multi-agent protocols, or a user-specified name. Use an externally-specified output path when provided; otherwise write to the current working folder.

**Artifact 2 — The chat run sheet (everything else).** Surface in the chat output, alongside (not inside) the prompt artifact: the standing environment assumptions, the deployment configuration block (when API-targeted), the Phase 5 delta analysis, and the Phase 6A QC self-audit summary. Present these as a session run sheet wrapping the prompt — clearly delimited so the user can read the run context in chat without contaminating what gets written to disk.

**The partition is absolute.** Even when the deliverable is a CLAUDE.md, environment and deployment context belong in the chat run sheet, not the file. The prompt body in the file matches exactly what would be in the chat prompt block — same content, no additions when written to disk.

## Standing Environment Assumptions

Every optimized prompt this skill produces assumes the downstream session runs under the following environmental defaults. Phase 4 surfaces these in the chat run sheet alongside the prompt — never injected into the prompt body and never written to the file. Do not omit them from the chat output and do not contradict them with cost-optimization, model-routing, or tool-allowlist guidance from anywhere else in this skill or its references.

- **Subscription tier:** Claude Max 20x. No rate limits, no usage caps, no token-budget constraints.
- **Context:** Full available context window. The downstream session does not need to conserve tokens or stop tasks early due to budget concerns.
- **Model:** Claude Opus 4.7 (`claude-opus-4-7`) with adaptive thinking for the orchestrator AND for every subagent. No exceptions — including long-context synthesis (use partitioning into sub-windows under 256K instead of model swap).
- **Subagent routing (Claude Code CLI):** `CLAUDE_CODE_SUBAGENT_MODEL=claude-opus-4-7[1m]`. Subagents inherit the same model as the orchestrator.
- **Tool selection:** Adaptive thinking decides which tools to invoke. Do not pre-specify a manual tool allowlist in optimized prompts. Where a task obviously requires a specific tool (research needing current data → web search; document extraction → file tools), instruct that tool's use directly. For ambiguous cases, leave it to the model's adaptive judgment.

When references in this skill recommend Sonnet/Haiku routing, Opus 4.6 fallbacks for long context, or cost-driven partitioning, treat those recommendations as superseded by this section. The standing environment is non-negotiable absent an explicit user override during Phase 1.

## References Guide

Load these on demand — do not pre-read all references. Each phase of the protocol points to the references it needs.

- **[references/opus-4-7-config.md](references/opus-4-7-config.md)** — Opus 4.7 critical behaviors, API parameters, effort levels, adaptive thinking, structured outputs, context window options, long-context verification posture.
- **[references/calibration-protocol.md](references/calibration-protocol.md)** — Phase 1.5 query set, source trust hierarchy, delta format, integration rules, fallback chain.
- **[references/platform-baseline.md](references/platform-baseline.md)** — Capability matrix across API, CLI, Cowork, claude.ai; subscription tiers; platform assignment guidance.
- **[references/inference-heuristics.md](references/inference-heuristics.md)** — Phase 0 Step 0C signal-to-inference lookup; orchestrated-research sniff rules; confidence calibration.
- **[references/landscape-research.md](references/landscape-research.md)** — Phase 2 operational manual: single-threaded vs orchestrated-fan-out execution modes, 2A draft deconstruction, 2B query taxonomy and source-type diversity, 2B' adversarial pass, 2C synthesis, 2D/2D' verification gates.
- **[references/task-heuristics.md](references/task-heuristics.md)** — Per-task-type guidance (code, data, research, writing, agent, orchestrated, creative, multi-modal, API, multi-model, knowledge work) plus the Opus 4.7 anti-patterns table.
- **[references/orchestrated-research.md](references/orchestrated-research.md)** — Complete multi-agent guide: decomposition, subagent construction, dependency management, synthesis, failure modes.
- **[references/prompt-template.md](references/prompt-template.md)** — Standard and orchestrator templates; embedding-without-bias rules.
- **[references/qa-checklist.md](references/qa-checklist.md)** — Phase-by-phase QA gate used by the Phase 6A self-audit.
- **[references/best-practices.md](references/best-practices.md)** — User-facing prep tips, standard vs. orchestrated variants, common pitfalls.

## Opus 4.7 Configuration & Critical Behaviors

For the full Opus 4.7 Critical Behaviors table (what carries forward from 4.6, what's new in 4.7), API parameters, effort levels (low/medium/high/xhigh/max), adaptive thinking syntax, the 128K output ceiling, streaming requirements, structured outputs (prefilling is removed on 4.7), context window options, and the long-context verification posture (1M context is usable; high-stakes exact-match recall should be user-validated against the target workload), see [references/opus-4-7-config.md](references/opus-4-7-config.md).

## User Preferences Do Not Override This Protocol

If the user's system prompt, user preferences, or conversation context includes instructions to "execute directly," "skip clarifying questions," "don't ask unnecessary questions," or any similar directive favoring speed over interaction, those preferences do not apply to this skill's interaction gates. The protocol phases exist because the user built them into the skill deliberately — they are the user's standing instructions for how prompt optimization should work, and they take precedence over general conversation-level preferences. Every widget-based phase fires on every run. The perceived clarity or completeness of the input spec is not a basis for skipping phases. Surface all findings from research phases (1.5, 2, 2.5) to the user so the resulting prompt reflects the user's informed choices, not the optimizer's assumptions about what the user would choose.

## Planner Anti-Collapse

**Judgment within phases is expected. Judgment across phases is forbidden — with one exception: track selection.**

*Within phases* — exercise judgment freely on question phrasing, option ordering, batch composition, synthesis depth, where to focus a search query, how to summarize calibration findings, which anti-patterns are most relevant to a given draft. The protocol does not over-specify intra-phase work because the model's judgment is load-bearing there.

*Across phases* — **do not condense, collapse, reorder, or skip phases on the basis of your own assessment that a use case is "clean," "simple," or otherwise doesn't warrant the full sequence.** If you find yourself drafting language that shortens the interaction flow ("rather than running the full sequence", "skipped for efficiency", "Execution Mode") — stop and run the selected track in full. The instinct to silently condense is the failure mode this rule exists to prevent.

*The one sanctioned form of right-sizing — track selection* — happens at **Phase 0.5**, against published objective criteria, and is surfaced to the user for confirmation before any phase is skipped. Phase 0.5 is not the planner exercising discretion to cut corners; it is the planner running a defined classification step whose output the user sees and can override. Once a track is selected and confirmed, **phase order, phase presence, and widget-call hard stops within that track are not negotiable.** The Express track is a different published sequence, not a license to improvise.

## Optimization Protocol

This protocol is a sequential state machine. Each phase depends on outputs from prior phases. Execute phases in order. Phase 0.5 triage selects one of two published tracks (Full or Express); within the selected track, do not produce Phase 4 output until all of that track's prior phases are complete.

### Turn-by-Turn Execution Plan

The one-widget-call-per-turn rule and the plain-text fallback are defined canonically in the Widget Interaction Protocol below. The table here shows how the phases map onto turns. Turn 1 always runs Phase 0.5 triage; the rows after it depend on the selected track.

**Full track:**

| Turn | Actions | Terminator |
|------|---------|-----------|
| 1 | Execute Phase 0 (intent extraction, requirement decomposition, Step 0C inferred answers). Execute Phase 0.5 triage. Present Phase 0 restatement + track decision in prose. Issue ONE `ask_user_input_v0` call with Widget Call 1 — core parameters (task type, deployment target, primary outcome). | Hard stop. |
| 2 | Record Turn 1 selections. Issue ONE `ask_user_input_v0` call with Widget Call 2 — constraints (failure modes, output format). | Hard stop. |
| 3 | Record Turn 2 selections. If conditional widgets are needed (orchestrated-research parameters 3a, or API deployment config 3b), issue ONE `ask_user_input_v0` call for the applicable conditional set. If both conditionals apply, issue 3a in this turn and 3b in the next. If no conditionals apply, skip to Turn 4 work. | Hard stop (if conditional) or pass-through. |
| 4 | Execute Phase 1.5 (web research for platform calibration). Execute Phase 2 (landscape research) — in 2-O mode this single step expands into a dispatch-and-collect cycle (parallel subagents → findings packs → 2C synthesis + 2D/2D') before the Phase 2.5 widget fires; the one-widget-per-turn rule is unaffected because subagent dispatch uses tool calls, not widget calls. Execute Phase 2.5 — if research-informed questions exist, present a brief prose summary of calibration findings, then issue ONE `ask_user_input_v0` call with the first batch of 1-3 Phase 2.5 questions. If Phase 2.5 was skipped, proceed directly to final output. | Hard stop (if widget issued) or pass-through. |
| 5+ | Phase 2.5 continuation (if needed) — ONE widget call per turn with the next batch until all Phase 2.5 questions are answered. | Hard stop per widget. |
| Final | Execute Phase 3 (draft analysis). Execute Phase 4 (produce optimized prompt). Execute Phase 5 (delta analysis). Execute Phase 6 (final QC pass + file write). Deliver the complete optimized prompt, delta analysis, and QC summary. | Done. |

**Express track** (selected at Phase 0.5 for mechanical / self-contained drafts):

| Turn | Actions | Terminator |
|------|---------|-----------|
| 1 | Execute Phase 0 and Phase 0.5 triage. Present Phase 0 restatement + track decision in prose. Issue ONE `ask_user_input_v0` call — the consolidated 1-Express widget (up to 3 questions: task type, deployment target, primary outcome; Low-confidence Step 0C items promoted in). | Hard stop. |
| 2 | Record selections. Execute Phase 1.5 (calibration — never skipped). If calibration surfaces a genuine unresolved ambiguity, issue ONE Phase 2.5 widget call for that ambiguity only; otherwise pass through. | Hard stop (if widget issued) or pass-through. |
| Final | Execute Phase 3 → 4 → 5 → 6. Deliver the optimized prompt, delta analysis, and QC summary. Phase 2 and (usually) Phase 2.5 are marked `N/A — Express track` in the QC summary. | Done. |

Phases marked with a hard stop are user-interaction gates. Present clickable multiple-choice widgets using the `ask_user_input_v0` tool, then stop and wait. Do not continue to the next phase until the user has responded. Every question presented to the user must use the widget format — never ask bare prose questions that require the user to type free-form answers.

### Non-Interactive Execution Mode

In environments where no user is reachable (subagent runs, scheduled tasks, CLI `-p` mode, headless automation), the widget gates defined in the Widget Interaction Protocol cannot be honored. In that mode only, fall through:

- **Phase 0 / 0.5:** treat Step 0C inferred answers as accepted (no widget confirmation); record their Signals + Confidence stamps as the working selections.
- **Phase 1 / 2.5:** collapse all widget gates; use the inferred values directly.
- **Phase 1.5:** still mandatory; surface findings in the chat run sheet equivalent (whatever the parent context receives as the subagent's output).
- **Phase 2 mode:** 2-O requires the Agent/Task tool, so a headless/toolless run that lacks it uses 2-S by definition (see Phase 2 § execution modes). A non-interactive run that *does* have the Agent tool may still fan out into 2-O.
- **Phase 5 delta:** explicitly list which answers were assumed and their Confidence stamps, so the user sees what was inferred without their input.
- **Phase 6A QC:** mark the run as non-interactive in the audit; the PASS/FAIL gates still apply.

This mode is the **only** sanctioned form of widget bypass — it does not authorize speed-driven user preferences to skip widgets. See "User Preferences Do Not Override This Protocol" above for the distinction.

### Year-Token Runtime Substitution

Do not rely on any hardcoded year or month inside this skill file or its references when constructing time-sensitive queries (calibration searches, landscape research) or outputs (citations, "current as of" dates). Use the current month and year as provided in this session's context. References that show bracketed tokens like `[current year]` or `[current month]` are literal substitution placeholders — replace them with the session's real current date at query-construction time.

### Model-Version Runtime Substitution

Where this skill names `Opus 4.7`, `claude-opus-4-7`, or any specific model identifier, those names are templated against the calibration-stamp date at the top of each reference file. **The substitution rule fires only when Phase 1.5 surfaces a confirmed flagship change via the broadened Query 4** (see `references/calibration-protocol.md`). In the absence of such a confirmed finding, use the model identifiers named in this skill verbatim — do not substitute speculatively, do not pre-emptively swap based on prior knowledge, do not guess at a newer flagship. When Phase 1.5 does confirm a newer flagship, use that newer ID for the optimized prompt's deployment configuration and re-run Query 4 against the new model card; do not preserve a retired model ID in the chat run sheet's `<deployment_config>` block.

### Widget Interaction Protocol

All clarifying questions in Phases 1 and 2.5 use the `ask_user_input_v0` tool when available. This tool presents clickable multiple-choice options to the user as an interactive widget at the bottom of the chat.

**One call per turn — enforced by the platform.** The `ask_user_input_v0` tool terminates the assistant's turn upon invocation. Any subsequent tool calls or text output in the same turn are silently dropped by the runtime. Issue exactly ONE `ask_user_input_v0` call as the last action of any turn that uses it. If you have 6 questions to ask, spread them across 2-3 turns (up to 3 questions per call, one call per turn). Never attempt multiple sequential widget calls in a single turn — only the first will render.

**Platform detection:** Before the first widget call, check whether `ask_user_input_v0` is available in your current tool set. If it is NOT available (Claude Code CLI, API, or any non-widget environment), use the plain-text fallback described below instead. If a widget call is made but fewer questions render than were included in the call, re-present the missing questions immediately as plain-text numbered options in your next response and wait for the user's text reply before proceeding.

**Plain-text fallback (non-widget platforms):**
When `ask_user_input_v0` is unavailable, present each question as a numbered list with lettered options. Example:
```
1. Task type — What best describes this prompt's primary function?
   a) Code generation
   b) Analysis / research
   c) Writing / content
   d) Other (please specify)

2. Deployment target — Where will this prompt run?
   a) Claude API
   b) Claude Code CLI
   c) claude.ai (chat)
   d) Other (please specify)
```
Wait for the user to reply with their selections (e.g., "1a, 2b") before proceeding. All hard-stop rules still apply — do not continue past an interaction gate until the user responds.

**Tool constraints (when widgets are available):**
- 1-3 questions per tool call
- 2-4 options per question
- Question types: `single_select` (pick one), `multi_select` (pick one or more), `rank_priorities` (drag-and-drop ranking)

**Option design rules:**
- Every question must include a write-in escape hatch as the final option: "Other (I'll specify)" or "None of the above — I'll clarify"
- Options should be short labels (2-8 words). Add a one-line description only when the label alone is ambiguous.
- Claude populates options using inferences from Phase 0 (for Phase 1) or research findings (for Phase 2.5). Options should represent Claude's best-informed hypotheses about the user's likely answer, not generic placeholders.
- Prefer `single_select` by default. Use `multi_select` only when the user genuinely might select multiple (e.g., failure modes — a single draft can have several at once). Use `rank_priorities` sparingly — only when relative ordering matters.

**Batching strategy:**
- Group related questions into the same tool call (max 3 per call)
- Present the most consequential questions first (task type and deployment target before output constraints)
- If a question is genuinely unbounded and cannot be reduced to 2-4 meaningful options even with Phase 0 inference, present it as a confirmation widget: Claude's best-guess answer as Option 1, one plausible alternative as Option 2, and "Other (I'll specify)" as Option 3

### Phase 0: Core Intent Extraction

**Mandatory regardless of input quality.** Extract the true objective before any other work.

If the user's draft appears thin (one sentence, no concrete artifact, no platform mention), reference the user to [references/best-practices.md](references/best-practices.md) for prep tips and the quick self-diagnostic before running additional widget rounds — a richer draft routes through Phase 0 with higher confidence and fewer interaction turns.

#### Step 0A: Identify the Atomic Task

Parse the user's input to answer: **What single artifact or outcome must this prompt produce?**

| Input Signal | Extraction Method |
|--------------|-------------------|
| Multiple conflicting instructions | Identify dominant theme; confirm with user |
| Vague goals ("help me with X") | Reframe as concrete deliverable |
| Stream-of-consciousness | Extract nouns + verbs; cluster by theme |
| Missing context | Infer likely use case; present assumption and confirm |
| Contradictory constraints | Surface contradiction; force prioritization |

#### Step 0B: Decompose into Testable Requirements

Convert to discrete, testable requirements:

```
CORE OBJECTIVE: [Single sentence — the artifact to be produced]
REQUIREMENTS:
- R1: [Requirement] → Success: [criterion] | Failure: [mode]
- R2: [Requirement] → Success: [criterion] | Failure: [mode]
UNSTATED ASSUMPTIONS: [Inferences made about user intent]
CONFIRMATION NEEDED: [Yes/No]
```

If CONFIRMATION NEEDED is Yes, prioritize confirming the core objective as Question 1 in Widget Call 1 before other clarifying questions.

#### Step 0C: Generate Inferred Answers for Phase 1

Before presenting Phase 1 questions, analyze the user's draft prompt to generate best-guess answers for every Phase 1 question. These inferences populate the multiple-choice widget options so the user can confirm with a click rather than typing from scratch.

For each Phase 1 question, produce a structured inference. **Every inference must include both `Signals:` and `Confidence:` — no exceptions.** The Signals trace cites what in the draft (or what absence in the draft) led to the inference. Confidence is High / Medium / Low based on how much the draft directly supports the inference vs. requires guessing.

```
INFERRED ANSWERS:
- Q1 (Task type): [Inferred type] | Signals: [What in the draft (or absence in the draft) suggests this] | Confidence: [High/Medium/Low]
- Q2 (Deployment target): [Inferred platform] | Signals: [Structural cues] | Confidence: [High/Medium/Low]
- Q3 (Primary outcome): [Best guess from draft] | Signals: [What in the draft supports this restatement] | Confidence: [High/Medium/Low]
- Q4 (Failure modes): [Top 2-3 likely failure modes based on task type and domain] | Signals: [Domain-driven or draft-stated] | Confidence: [High/Medium/Low]
- Q5 (Output constraints): [Inferred format/length from draft structure] | Signals: [Format cues in draft, or absence thereof] | Confidence: [High/Medium/Low]
```

**Confidence calibration.** When the draft is minimal (one sentence, no concrete artifact, no platform mention), default to Low — not Medium. Medium implies there's a concrete cue you're partially trusting; if the cue is absent, it's Low. Multiple Low-confidence inferences are a signal to widen Widget Call 1 options to all 4 slots.

For a worked example showing INFERRED ANSWERS populated for a representative draft, see [references/inference-heuristics.md § Worked example](references/inference-heuristics.md#worked-example).

**Orchestrated-research sniff.** During Step 0C, inspect the draft for orchestrator signals ("parallel agents," "subagents," "dispatch," "synthesize," "wave 1," "fan out," Agent tool references, agent-count mentions, CLAUDE.md combined with dispatch language). When signals warrant, set the Phase 1 Widget Call 1 task-type default to `orchestrated-research` so it pre-selects in the widget. The widget itself remains — the user confirms or edits before Phase 2. Do not unilaterally decide task type or skip Phase 1.

**For the full signal-to-inference lookup table and confidence calibration rules, see [references/inference-heuristics.md](references/inference-heuristics.md).**

### Phase 0.5: Complexity Triage

**Purpose:** Right-size the protocol to the input so a trivial draft does not pay the cost of a full research-and-clarification sequence, and a substantive draft is never under-served. This is the one sanctioned form of cross-phase right-sizing (see Planner Anti-Collapse). It runs once, after Phase 0, before Widget Call 1.

Classify the draft into one of two tracks against the objective criteria below. Apply the criteria in order; the first matching rule decides the track. When criteria conflict or the draft sits on the boundary, default to the **Full track** — the asymmetry of cost favors over-serving.

**Full track** — the draft meets ANY of:

- The work involves factual claims, research, analysis, decision-support, or contested-domain content (anything Phase 2 landscape research would materially improve).
- The task type is `orchestrated-research`, `agent / workflow`, `research / search`, `data analysis`, or `multi-modal`.
- The draft spans multiple files, multiple sub-domains, or multiple stakeholders.
- The draft is longer than ~150 words, OR Phase 0 surfaced contradictory constraints, OR two or more Step 0C inferences came back Low confidence.
- The draft is API-targeted and needs a deployment-configuration block.

**Express track** — the draft meets ALL of:

- The task is purely mechanical or a self-contained utility: reformatting, a format conversion, a single-function code snippet with a complete spec, a tightly-scoped rewrite — with no factual claims that research would improve.
- The draft is under ~150 words and the spec is complete (no contradictory or missing constraints; Step 0C inferences are Medium-or-better across the board).
- No landscape research would change the output — there is no implicit thesis to steel-man and no contested domain.

**Track sequences:**

| Track | Phase sequence |
|-------|----------------|
| **Full** | 0 → 0.5 → 1 (Widget Calls 1, 2, conditional 3a/3b) → 1.5 → 2 → 2.5 → 3 → 4 → 5 → 6 |
| **Express** | 0 → 0.5 → 1-Express (ONE consolidated Widget Call) → 1.5 → 3 → 4 → 5 → 6 |

The Express track keeps Phase 1.5 (platform calibration is cheap and catches deprecated parameters — never skip it) and the full Phase 3–6 sequence (draft analysis, prompt construction, delta, QC, file write are not optional regardless of input size). It skips Phase 2 entirely and skips Phase 2.5 unless Phase 1.5 calibration surfaces a genuine ambiguity the draft does not resolve — in which case run a single Phase 2.5 widget call for that ambiguity only.

**1-Express — the consolidated widget call.** The Express track replaces the three-turn Widget Call 1/2/(3) sequence with ONE `ask_user_input_v0` call carrying up to 3 questions: task type + deployment target (combined or as the two most consequential), and primary outcome confirmation. Failure modes and output constraints are inferred from Step 0C and stated as documented assumptions in the Phase 5 delta rather than asked, unless a Step 0C inference for them came back Low confidence — promote any Low-confidence item into the consolidated widget call, displacing a lower-priority question.

**Surface the track decision to the user.** State the selected track and the one-sentence reason in the Phase 0 restatement prose ("This is a self-contained reformatting task with a complete spec — running the Express track: one round of questions, no landscape research."). The user can override by replying — an override to Full track is always honored; an override to Express is honored only if the draft does not meet a Full-track criterion that involves factual or research content. The research-bearing Full-track criteria are #1 (factual claims, research, analysis, decision-support, contested-domain content) and #2 (task type is orchestrated-research / agent / workflow / research / search / data analysis / multi-modal). Other Full-track criteria — word count, multiple stakeholders, low confidence — are not research-bearing and *can* be overridden.

**QC handling.** The Phase 6A self-audit records the track. Phases skipped by the Express track are marked `N/A — Express track` in the QC summary, not `FAIL`.

### Phase 1: Clarifying Questions (Widget-Based)

The phase below describes the **Full track** Widget Call sequence. On the **Express track**, replace Widget Calls 1/2/3 with the single 1-Express consolidated call defined in Phase 0.5, then proceed to Phase 1.5. In both tracks: present the Phase 0 restatement in prose first, populate option labels from the Step 0C inferred answers, and issue one widget call per turn per the Turn-by-Turn table above.

**Widget Call 1 — Core Parameters (Turn 1)** (3 questions, single_select):

1. **Task type**: Lead with the inferred type from Step 0C. Include orchestrated-research as an explicit option when signals are present. Example options: `["Orchestrated research (multi-agent CLI)", "Code generation", "Analysis / research", "Other (I'll specify)"]` — exact ordering reflects the Step 0C inference.
2. **Deployment target**: Lead with the inferred platform. Example options: `["Claude API", "Claude Code CLI", "Cowork (Desktop)", "claude.ai (chat)", "Other (I'll specify)"]`. Cowork (Desktop) is a first-class GA deployment target on all paid plans.
3. **Primary outcome**: Present Claude's inferred CORE OBJECTIVE as Option 1 (a short restatement), one alternative interpretation as Option 2, and "Other (I'll specify)" as Option 3. If CONFIRMATION NEEDED = Yes from Phase 0, this becomes Question 1 instead of Question 3.

**→ HARD STOP after Widget Call 1.** End the turn. Wait for selections.

**Widget Call 2 — Constraints & Failure Modes (Turn 2)** (2 questions):

4. **Failure modes** (multi_select): 2-3 likely failure modes inferred from task type and domain, plus write-in.
5. **Output constraints** (single_select): Inferred output format as Option 1, plausible alternatives 2-3, write-in.

**→ HARD STOP after Widget Call 2.**

**Widget Call 3a — Conditional: Orchestrated Research (Turn 3, if applicable)** (only if Widget Call 1 task type = orchestrated-research):

6. **Agent structure** (single_select): `["Fully parallel (all independent)", "Wave-based (some dependencies)", "Hybrid (shared constants + parallel)", "Other (I'll specify)"]`
7. **Remediation pass** (single_select): `["Yes — first pass, expect corrections", "No — targeted follow-up", "Other (I'll specify)"]`

**Widget Call 3b — Conditional: API Deployment (Turn 3 or Turn 4, if applicable)** (only if Widget Call 1 deployment target = API):

8. **Thinking mode & effort** (single_select): `["Adaptive thinking — high effort", "Adaptive thinking — xhigh effort", "Adaptive thinking — max effort", "Other (I'll specify)"]`
9. **Context scale** (single_select): `["Standard 200K", "1M context (GA)", "Other (I'll specify)"]`

When both 3a and 3b apply, issue 3a first, then 3b in the following turn. Never combine conditional widgets.

**→ HARD STOP after any conditional widget.** Only after all Phase 1 interactions are complete, proceed to Phase 1.5.

#### Handling User Selections

When the user selects "Other (I'll specify)," incorporate their free-text follow-up. For widget-selected answers, record directly into working requirements — no further confirmation.

If selections reveal a contradiction (e.g., task type = orchestrated-research but deployment target = claude.ai, which doesn't support subagents), surface the conflict briefly and issue one follow-up widget to resolve. That follow-up is the only widget call in its turn.

### Phase 1.5: Platform & Best Practices Calibration (Mandatory — Every Run)

**Purpose:** Ensure the optimized prompt reflects current prompt engineering best practices and platform-specific capabilities. The Opus 4.7 behavioral baseline in this skill is a starting point — Anthropic ships model updates, new API features, and revised documentation on a rolling basis. This phase catches what has changed since the skill was last edited.

For the full calibration query set, source trust hierarchy, delta format, integration rules, confidence levels, and fallback chain, see [references/calibration-protocol.md](references/calibration-protocol.md).

**Step 1.5A — Deployment target identification.** Confirmed in Phase 1. If ambiguous, issue one clarifying widget before proceeding.

**Step 1.5B — Targeted best-practices research.** Run the 4 mandatory queries below, scoped to authoritative sources (docs.anthropic.com, anthropic.com/news, model cards, platform guides). At least one query must target features unique to the deployment target. Surface 2+ platform-unique features in the delta.

| # | Query template | Purpose |
|---|---|---|
| 1 | `Claude Opus 4.7 [deployment target] best practices [current year]` | Platform-specific recent guidance |
| 2 | `Anthropic prompt engineering [task type] [feature mentioned in draft]` | Task-type-specific guidance |
| 3 | `Claude Opus 4.7 [deployment target] configuration` | Deployment-target API/setup specifics |
| 4 | `Anthropic Claude model card Opus 4.7` | Model-card capabilities/limitations |

**Query 5 (conditional)** — Fire if any of: (a) draft references a feature not surfaced by Queries 1–4 (MCP, Files API, Batch API, custom tools, prompt caching, Skills, Cowork, Managed Agents); (b) baseline appears stale (>30 days since last calibration evidence); (c) Queries 1–4 surface a model retirement notice or deprecation. Query 5 form: `Anthropic [specific feature] [deployment target] [current year]`.

**Step 1.5C — Current practices delta.** Produce a structured delta: new/updated practices, platform-specific considerations, deprecated/superseded patterns, no-change confirmations. If research surfaces no meaningful changes, state that explicitly — confirming baseline is itself valuable signal. Do not fabricate deltas.

**Step 1.5D — Integration gate.** Verify delta findings are factual and sourced, platform constraints captured, deprecated features flagged for Phase 3 check, delta ready to feed Phase 4 prompt construction.

### Phase 2: Landscape Reconnaissance

**Purpose:** Identify the full scope of topics, considerations, and edge cases the prompt must address — WITHOUT prejudging conclusions.

**Mandatory** for any prompt involving factual domains, research, analysis, or decision-making. Skip only for purely mechanical tasks (reformatting, simple code generation with complete specs); when skipping, set the skip flag explicitly so Phase 3 and the QA checklist treat Phase 2 as N/A rather than incomplete.

**Critical Phase 2 constraint.** Research identifies WHAT AREAS TO COVER. It must NEVER influence WHAT CONCLUSIONS the final prompt should reach. The optimized prompt instructs the executor to research the identified areas; it does not bake research conclusions into the prompt as assumptions. *Topic vs. position:* permissible — "Cover Basel III endgame proposals and their disputed CRE-lending impacts; present both sides." Not permissible — "Conclude that Basel III endgame will reduce CRE lending capacity." When in doubt, frame as "address X," not "conclude Y about X."

**Execution mode — single-threaded or orchestrated.** Phase 2 runs in one of two modes. Both execute the same logical steps (2A → 2B → 2B' → 2C → 2D → 2D'), honor the same topic-not-position discipline, and produce the same 2C synthesis artifact and 2D/2D' gate results — they differ only in *who* runs the searches and *how much depth* the corpus reaches:

- **Phase 2-S (single-threaded)** — the orchestrator runs every step itself under the 12-search hard cap. This is the default and the only mode available when the Agent/Task tool is absent (claude.ai chat, API without a dispatch harness).
- **Phase 2-O (orchestrated fan-out)** — the orchestrator runs 2A itself, then dispatches parallel landscape subagents (one per sub-domain or query-taxonomy lane, plus a dedicated adversarial-pass subagent), each searching deeply within its own 1M-context lane, then collects their findings packs and runs 2C → 2D/2D' on the aggregate corpus. Depth comes from parallel lanes and at most one remediation pass — not from unbounded recursion.

Select 2-O when ALL hold: (1) the Agent/Task tool is available in the current session; (2) the draft is genuinely multi-sub-domain, multi-stakeholder, or contested — the same depth signals that put a draft on the Full track. Otherwise run 2-S. The choice is made on these objective criteria, not planner discretion, and is reported (mode + one-line rationale) in the chat run sheet and the Phase 5 delta. A user override to 2-S is always honored; an override to 2-O is honored only when the Agent/Task tool is actually available. Both modes inherit the Standing Environment Assumptions (every subagent on `claude-opus-4-7[1m]` with adaptive thinking); subagents cannot spawn subagents; the neutrality discipline and the Deliverable Contract partition apply identically — in 2-O they are carried into every subagent brief and re-checked across the aggregate corpus at 2D/2D'.

**Operational detail lives in [references/landscape-research.md](references/landscape-research.md).** Load that file when Phase 2 runs. It carries the single-threaded vs orchestrated execution-mode procedure, the Step 2A draft deconstruction, the Step 2B query taxonomy and source-type diversity requirements, the Step 2B' adversarial pass, the Step 2C synthesis artifact, and the 2D / 2D' verification gates. The phase contract to hold in mind:

- **2A deconstructs the draft's vantage** (topical content, implicit framing, stakeholder map, contested terminology) so research targets the full landscape, not the draft's narrow slice.
- **2B executes structured search** — a query taxonomy (consensus / dissent / practitioner / academic / recent / failure-case / adjacent / non-mainstream) and a source-type diversity floor (3 of 5 categories). In 2-S this is capped at 12 total Phase 2 searches; in 2-O the same taxonomy and floor apply per lane, bounded by per-lane budgets instead.
- **2B' runs an adversarial pass at parity** — comparable search effort against the strongest counter-framing, so the position the draft most needs to confront actually surfaces.
- **2C synthesizes** the corpus into a structured artifact (sub-domains, frameworks, debates, temporal, edge cases, draft gaps), each item confidence-stamped.
- **2D + 2D' are parallel verification gates.** 2D checks position bias (findings must not become conclusions); 2D' checks coverage bias (narrow inputs producing a technically-neutral but actually-biased synthesis). Both must pass before Phase 2.5.

All Phase 2 artifacts are internal working machinery — they feed downstream phases and the chat run sheet, never the optimized prompt body or the written file (Deliverable Contract partition; the Phase 6B strip check enforces it).


### Phase 2.5: Research-Informed Clarification (Widget-Based)

**Purpose:** Use domain knowledge from Phases 1.5/2 to pressure-test stated intent with precise, informed questions — presented as widgets. Phase 1 captured structural parameters before research. Now that you understand the landscape — sub-topics, frameworks, controversies, edge cases, platform constraints — you can identify gaps, ambiguities, and unstated assumptions invisible earlier.

**When Phase 2 ran (most cases):** Research routinely surfaces considerations the user hasn't articulated. This phase is mandatory even when stated intent seems clear. Skipping risks building around a reasonable approximation of intent rather than actual intent.

**When Phase 2 was skipped (mechanical tasks):** Draw only from Phase 1.5 calibration findings. The 5-10 target does not apply — present however many material questions the platform delta supports (possibly zero). If platform calibration surfaces no ambiguities the draft doesn't resolve, skip this phase with a one-line note and document in Phase 5 delta.

**Step 2.5A — Generate research-informed questions.** For each Phase 1.5/2 finding, ask: does the user's stated intent resolve this, or is there an unstated assumption I'd be guessing at? Formulate 5-10 clarifying questions. Each must pass three gates:

1. **Research-grounded**: Could not have been asked before Phases 1.5/2.
2. **Intent-refining**: Answer will materially change the optimized prompt's content, structure, or constraints.
3. **Non-redundant**: Not already answered by Phase 0/1 responses or the draft.

For each question, generate 2-3 research-informed options plus a write-in escape. Options must be concrete — derived from actual research findings, not generic placeholders.

**Step 2.5B — Present questions as widgets.** Brief prose intro summarizing calibration findings (2-3 sentences), then issue ONE `ask_user_input_v0` call with the first batch (up to 3 questions). Remaining questions carry to subsequent turns — one widget call per turn. Each question text includes a one-sentence rationale naming the research finding that prompted it.

**Step 2.5B' — Batching when >3 questions surface.** Issue at most 3 questions per widget call. Prioritize by consequence:

1. **Scope-changing** first — questions whose answers add or remove sub-domains, change deployment constraints, or change output structure.
2. **Content-changing** second — questions whose answers change what the prompt instructs Claude to do, but not the overall shape.
3. **Stylistic / tone** third — questions about phrasing, formatting nuance, length preferences.

If >6 questions surface (more than 2 widget calls' worth), prefer deferring tier-3 questions to Phase 5 documented assumptions over expanding to 4+ widget turns. Long widget chains erode the user's energy for the questions that actually matter.

**→ HARD STOP.** Do not proceed to Phase 3 until all Phase 2.5 batches are answered. If Phase 2.5 was skipped, proceed directly.

**Step 2.5C — Integration.** Update Phase 0 requirements with new/refined requirements. Adjust Phase 2 landscape synthesis for scope decisions (in/out). Note selections that create new constraints for Phase 4. Record "Other (I'll specify)" free-text. Record deferred questions with explicit assumptions for Phase 5 delta.

**Step 2.5D — Confirmation gate.** Verify all widget selections incorporated, scope exclusions documented (suppresses Phase 3 gap flags for excluded sub-domains), deferred questions have explicit assumptions.

### Phase 3: Draft Analysis

Identify issues in these categories. Cross-reference against [references/task-heuristics.md](references/task-heuristics.md) for task-type-specific anti-patterns and missing enhancements.

**Structural gaps:** Missing success criteria, no output format spec, unclear task boundaries, missing context/motivation, coverage gaps from Phase 2 (as refined by Phase 2.5 scope decisions — do not flag sub-domains the user explicitly placed out of scope).

**Precision deficits:** Vague verbs ("help with" → "generate", "analyze", "extract"), implicit assumptions, ambiguous scope, undefined handling of controversies. Incorporate Phase 2.5 new constraints.

**Opus 4.7 anti-patterns.** Cross-reference the Phase 1.5 Current Practices Delta before applying this check. If calibration identified new anti-patterns or deprecated behaviors, fold them in. If calibration confirmed baseline is current, proceed as-is.

For the full Opus 4.7 anti-pattern table — emphasis discipline, deprecated parameters (`budget_tokens`, `output_format`, prefilling), guardrail gaps, orchestrator under-delegation, tool-use defaults — see [references/task-heuristics.md §Opus 4.7 Anti-Patterns](references/task-heuristics.md#opus-47-anti-patterns). Apply every row that matches the draft; flag in Phase 5 delta which rows triggered fixes.

**Comprehensiveness deficits (from Phase 2, as refined by Phase 2.5):** Missing sub-domains that remain in scope after user clarification, no debate-handling instructions for controversies confirmed relevant, undefined temporal scope, unaddressed edge cases.

### Phase 4: Produce Optimized Prompt

**Prerequisite check — verify all of the following before producing any output:**
- Phase 0 core objective, requirements, and Step 0C inferred answers documented
- Phase 1 widget-based questions presented; user selections recorded
- Phase 1.5 web research executed; calibration delta produced
- Phase 2 landscape research executed (or explicitly skipped for mechanical tasks)
- Phase 2.5 widget-based questions presented; user selections recorded (or explicitly skipped)
- Phase 3 draft analysis identified structural gaps, precision deficits, and anti-patterns

If any prerequisite phase was not executed, return to the earliest incomplete phase. Do not produce Phase 4 output from a cold start.

The optimized prompt must be: (1) RUNNABLE — no unresolved placeholders, (2) COMPREHENSIVE — all landscape findings as areas to cover, (3) NEUTRAL — zero pre-judged conclusions.

**Template selection.** Read [references/prompt-template.md](references/prompt-template.md) for the full standard template and orchestrator template. Select the appropriate template based on the Phase 1 task type. For orchestrated-research, the deliverable is a CLAUDE.md file structured per [references/orchestrated-research.md](references/orchestrated-research.md) Section 10.

**Partition enforcement at template application.** When the template includes `<environment>` or `<deployment_config>` blocks, those blocks are rendered into the chat run sheet only — never into the prompt body that becomes the file. The prompt body and the file body are identical; both exclude environment and deployment metadata. This applies to every task type, including CLAUDE.md.

**Embedding landscape insights without bias** (full 10-row table with controversial-domain language guidance in [`references/prompt-template.md` § Embedding landscape insights without bias](references/prompt-template.md#embedding-landscape-insights-without-bias)):

| Permissible | Not Permissible |
|-------------|-----------------|
| "Address sub-topics: A, B, C" | "Topic A is more important than B" |
| "Sources disagree on X — present both views" | "The correct view on X is…" |

### Phase 5: Delta Analysis

After producing the optimized prompt, deliver alongside it:

1. **Key changes** — Bullet list of modifications with rationale
2. **Calibration-driven changes** — What was updated from Phase 1.5 research (or baseline-held)
3. **Landscape-driven additions** — What was added from Phase 2 research. Note which execution mode ran (2-S single-threaded or 2-O orchestrated fan-out) and the one-line rationale; for 2-O, name the subagent lanes dispatched and whether a remediation pass fired.
4. **Clarification-driven refinements** — What was added/removed/changed from Phase 1 and Phase 2.5 widget selections. Flag "Other (I'll specify)" responses and how integrated. Flag deferred questions and assumptions made.
5. **Neutrality attestation** — No pre-judged conclusions embedded
6. **Risk flags** — Potential failure modes to monitor
7. **Iteration hooks** — What to tune if output underperforms

### Phase 6: Final QC Pass + Deliverable File

#### Step 6A: Self-Audit QC Pass

Re-run through the skill phases as a verification sweep. For each phase, confirm that its outputs were properly incorporated into the final deliverable:

1. **Phase 0 verification**: Core objective and decomposed requirements faithfully reflected — no drift from original intent
2. **Phase 1 verification**: Every widget selection (task type, deployment target, primary outcome, failure modes, output constraints, conditional selections) explicitly addressed in the output
3. **Phase 1.5 verification**: All calibration delta items (new capabilities, changed behaviors, deprecated patterns) reflected. No stale patterns remain.
4. **Phase 2 verification** (if applicable): All landscape research findings incorporated. Comprehensiveness gaps closed.
5. **Phase 2.5 verification** (if applicable): All research-informed clarification selections addressed
6. **Phase 3-4 verification**: Draft analysis issues resolved in the final output. Task-type heuristics applied.
7. **Phase 5 verification**: Delta analysis accurate and complete

**If gaps are found, classify and act:**

- **Material gap** — missing Phase 1.5/2 research finding from the output, ignored Phase 1 or 2.5 widget selection, stale 4.6-era pattern present, missing required template section, missing deployment configuration block from the chat run sheet on an API-targeted prompt (the deployment config does not go in the file — verify it is present in the chat run sheet only), missing completion-verification on a file-writing agentic prompt, any Opus 4.7 anti-pattern from `task-heuristics.md` left unaddressed, OR any environment/deployment/QC content that has leaked into the written file body. Fix and document the change in the QC summary.
- **Minor gap** — stylistic inconsistency, capitalization nit, redundant phrasing, formatting variance, comment polish. Fix silently.

If unsure which tier a gap belongs to, treat as material — the asymmetry of cost favors documentation.

**If no gaps are found:** Confirm a clean QC pass with a brief summary using the format below.

**Required QC Summary artifact** — use the format defined in [`references/qa-checklist.md` § QC Summary template](references/qa-checklist.md#qc-summary-template). It lives in the chat run sheet only, never in the written prompt file.

#### Step 6B: Write the Deliverable File

After the QC pass, write the finalized optimized prompt to an output path. Use the externally-specified path if one was provided; otherwise write to the current working folder with a descriptive filename (`optimized-prompt.md`, `CLAUDE.md`, `orchestrator.md`, or user-specified).

**Pre-write strip check (mandatory).** Before writing, verify the file content contains ONLY the prompt body. Specifically, confirm none of the following are present in what is about to be written:
- `<environment>` block or any environment metadata
- `<deployment_config>` block or any deployment metadata (thinking mode, effort, context window, compaction, output ceiling, streaming)
- QC self-audit summary, table, or pass/fail attestation
- Phase 5 delta analysis or change-log narrative
- Frontmatter comments documenting the run (audit date, calibration date, generation metadata)
- Preamble like "Here is the optimized prompt:" or postamble like "Saved to:" inside the file body
- Any "Generated by prompt-optimization skill" attribution
- Phase 2 internal working artifacts: the 2A Topic Identification & Draft Deconstruction block, the 2B Landscape Research Execution Log, the 2C Landscape Synthesis output, the 2D' Coverage-Bias Check, the 2-O subagent briefs / dispatch graph / per-lane findings packs, or any other Phase 2 analytical scaffolding

If any of the above are present in the proposed file content, strip them before writing. The file body must match exactly the prompt body that appears in the chat output's prompt block — same content, no additions, no metadata.

**Confirm and report.** Confirm the file was written with the folder-relative path in the chat response. The chat run sheet (prompt + environment + deployment config + delta + QC summary) remains the full deliverable in chat; only the prompt body travels to the file.

**Environment handling:** If the environment is claude.ai web chat and file writing is unavailable, skip the file-writing step and note to the user that they can copy the inline prompt block manually (only the prompt block — not the surrounding run sheet). Do not skip the inline deliverable under any circumstances.

## Task-Type Heuristics

Apply enhancements based on task type. See [references/task-heuristics.md](references/task-heuristics.md) for detailed Opus 4.7 guidance on code generation, data analysis, research/search, writing/content, agent/workflow, orchestrated multi-agent research, creative, multi-modal, API/integration, and knowledge work (Cowork-primary).

## Orchestrated Multi-Agent Research

For prompts targeting Claude Code CLI with parallel subagent execution (task type `orchestrated-research`), see [references/orchestrated-research.md](references/orchestrated-research.md) for the complete operational guide: subagent prompt construction, context isolation, dependency management (parallel/sequential/shared constants), output scope calibration, synthesis design, the remediation pattern, environment configuration, the orchestrator prompt template, and the failure mode catalog. See Meta-Rule 13 below for the Opus 4.7 dispatch requirement.

## Meta-Rules

1. **Execute the selected track in full** — see Planner Anti-Collapse above. Within the selected track: one `ask_user_input_v0` call per turn; Phase 1.5 fires on every run; Phase 2/2.5 fire on Full track for factual / research / decision-support content; do not proceed to Phase 4 until all of the selected track's prior phases are complete.
2. **Neutrality is paramount.** The optimized prompt defines WHAT TO COVER, never WHAT TO CONCLUDE.
3. **Optimized prompts must be RUNNABLE** — no placeholders unless explicitly templated.
4. **Preserve user's domain terminology exactly.**
5. **If draft is already near-optimal, say so** — don't over-engineer.
6. **For JSON output:** use `output_config.format` with JSON schema; specify "output ONLY valid JSON with no preamble" as fallback.
7. **For reasoning tasks:** specify effort level (low/medium/high/xhigh/max). If adaptive thinking is enabled, Claude decides when to reason deeply. Avoid "think" in prompts where thinking is NOT enabled.
8. **Distinguish comprehensiveness from bias:** "Cover topics A, B, C" is comprehensive; "conclude that A > B" is biased.
9. **When in doubt, make the prompt ask.** Instruct the executor to investigate and decide rather than embedding your judgment.
10. **Strip aggressive prompting language.** Neutral imperatives outperform emphasis markers on Opus 4.7.
11. **Include guardrails for code and agentic prompts.** Over-engineering guardrails for code; reversibility guardrails for agentic deployments; completion-verification for anything writing to disk (4.7 occasionally misreports completion).
12. **Specify deployment configuration** for any API-targeted prompt — surface it in the chat run sheet alongside the prompt artifact, not inside the prompt body and not in the written file. Cover: thinking mode, effort, context window, compaction, output ceiling, streaming.
13. **For orchestrated research:** Read [`references/orchestrated-research.md`](references/orchestrated-research.md) before constructing multi-agent prompts. Two invariants the orchestrator prompt must enforce — (1) explicit subagent dispatch instruction (4.7 under-delegates by default); (2) never embed unverified numerical estimates in subagent prompts (use shared-constants pattern or instruct independent verification).
14. **Instruct tool use when the task obviously requires a specific tool** (e.g., research that requires current data → web search; document extraction → file tools; agentic file modification → Edit/Write). For ambiguous cases, let Opus 4.7's adaptive thinking decide which tools to invoke. The goal is task-driven tool instruction, not a manual allowlist. Do not embed pre-specified tool inventories in the optimized prompt.

## Quality Assurance Checklist

Before delivering Phase 4 output, read and verify every applicable section in [references/qa-checklist.md](references/qa-checklist.md). Phase 6 (Final QC + file write) runs after delivery as the closing step.

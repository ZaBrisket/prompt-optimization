# Inference Heuristics — Step 0C Signal Lookup, Sniff Rules, Confidence Calibration

> **Calibration stamp:** This reference is skill-internal methodology (Step 0C inference heuristics), not Anthropic product facts — it does not require Phase 1.5 re-verification.

This reference is loaded by Phase 0 Step 0C, which generates inferred answers for every Phase 1 question before the widget call. The point is to populate widget options with the user's *likely* answers so they can confirm with a click rather than typing — not to bypass the widget.

## Operating principle

Every Phase 1 inference must record three things in the structured output: the inferred value, the **Signals** that produced it (what in the draft, or what *absence* in the draft), and the **Confidence** (High / Medium / Low). Inferences without signals are guesses; inferences without confidence stamps cannot be calibrated by downstream phases.

The signal lookup tables below pre-digest the most common cues for each Phase 1 question. They are not exhaustive — the model's judgment on novel signals is expected and welcome. When a signal isn't in the table, name it explicitly in the `Signals:` field of the inference output anyway, so the reasoning is auditable.

## Question 1 — Task type

| Signal in the draft | Inferred task type | Default confidence |
|---------------------|--------------------|--------------------|
| Code, programming language names, library names, file paths, function names | Code generation | High |
| "Analyze," "evaluate," "interpret," numerical data references, charts, statistics | Data analysis | High when data is explicit; Medium when implied |
| "Research," "find out," "what is the current state of," topic-broad queries with no clear deliverable shape | Research / search | Medium |
| Long-form narrative shape, audience reference, voice or tone references | Writing / content | High when the deliverable is a document; Medium for generic "write about X" |
| Agentic verbs ("modify," "deploy," "run," "execute," "fix"), file-modification references, multi-step workflow language | Agent / workflow | High when modification is named; Medium for "automate this" |
| **See orchestrated-research sniff rule below** | Orchestrated multi-agent research | High when ≥2 sniff signals present; Medium for 1 |
| Voice / persona references, genre names (poem, story, script), constraint references (sonnet, haiku, dialogue) | Creative | High |
| Image, screenshot, diagram, video, audio references | Multi-modal | High |
| `requests.post(...)`, "the API," request/response shape language, JSON schema references | API / integration | High |
| References to multiple Claude models, routing logic, "use X for Y and Z for W" | Multi-model | High when model names appear; Medium when "different models" is generic |
| File-folder navigation, document editing in Word/Excel/PowerPoint, persistent memory across sessions, desktop-app references | Knowledge work (Cowork-primary) | High when Cowork is named; Medium when desktop file-work is implied |
| **None of the above clearly matches; the draft is one sentence with no concrete artifact** | Best guess from the strongest cue; widen Widget Call 1 to 4 slots | **Low** (always) |

### Orchestrated-research sniff rule

Count signal hits from the list below, then apply the 3-tier rule.

**Signals to count:**

- "Parallel agents" / "subagents" / "parallel subagents"
- "Dispatch" / "fan out" / "fan-out" / "fanout"
- "Synthesize" / "synthesis" / "synthesizer"
- "Wave 1" / "wave-based" / "stage 1 / stage 2" sequencing language
- `Agent` tool references / `Task` tool references / `Task()` syntax
- Agent-count mentions ("3 agents," "five subagents," "N agents")
- `CLAUDE.md` references combined with dispatch language
- `CLAUDE_CODE_SUBAGENT_MODEL` references / subagent model configuration
- `.claude/agents/` or `~/.claude/agents/` path references
- "Orchestrator" / "orchestration" used as a noun

**3-tier rule:**

| Signal count | Widget Call 1 placement | Confidence |
|--------------|--------------------------|------------|
| 0 | Orchestrated-research not in lead options unless task type maps to it via other cues | — |
| 1 | Orchestrated-research as Option 2 (not lead); next-best task-type leads | Medium |
| 2+ | Orchestrated-research as Option 1 (lead) | High |

The widget itself always fires; the user confirms or edits.

**False-positive guard.** A signal phrase counts only when paired with a CLI / multi-instance / dispatch context cue (Claude Code CLI mention, `Agent` / `Task` tool reference, `CLAUDE.md`, concrete multi-instance dispatch infrastructure). Common false positives that don't count without that context: "synthesize the findings" in a single-thread research brief, "dispatch" in messaging contexts, "wave" in temporal/historical prose ("first wave of adoption"), "Agent" capitalized as a person's role ("the Agent for the deal"), "orchestration" as a metaphor ("orchestration of the team"). In doubt, downgrade the signal count.

## Question 2 — Deployment target

| Signal in the draft | Inferred target | Default confidence |
|---------------------|-----------------|--------------------|
| "API," `requests.post(...)`, model ID strings in code form, request/response examples | Claude API | High |
| `claude -p`, CLI invocations, `CLAUDE.md`, `.claude/` paths, subagent dispatch | Claude Code CLI | High |
| Cowork name reference, "desktop," persistent memory across sessions, file-folder workflows | Cowork (Desktop) | High when Cowork named; Medium when desktop-shape inferred |
| "Chat," "in claude.ai," interactive single-conversation shape, no file write expected | claude.ai (chat) | Medium (chat is the default fallback when no other signal is present) |
| **Multiple signals, e.g., orchestrated-research + Cowork** | Surface the conflict — orchestrated-research requires CLI | Low confidence on either; route through a follow-up widget |
| **No platform signal at all** | Lead with the platform most consistent with task type (e.g., orchestrated-research → CLI; knowledge work → Cowork) | **Low** |

## Question 3 — Primary outcome

Inference is the model's restatement of the user's CORE OBJECTIVE from Phase 0 Step 0A. Confidence calibration:

| Source of restatement | Default confidence |
|-----------------------|---------------------|
| The user wrote a concrete objective verbatim or near-verbatim | High |
| The user wrote a vague goal ("help with X"); the model inferred the artifact shape from domain cues | Medium |
| The user wrote a stream-of-consciousness draft; the model clustered nouns/verbs and named the dominant theme | Low |
| Multiple conflicting objectives present in the draft | Low (route through CONFIRMATION NEEDED → Question 1 of Widget Call 1) |

Widget Call 1 Question 3 presents Claude's inferred restatement as Option 1, one plausible alternative interpretation as Option 2, and "Other (I'll specify)" as Option 3. When confidence is Low, lead with the most-likely interpretation but make Option 2 a meaningfully different alternative.

## Question 4 — Failure modes

Inferred from task type and domain, not from explicit draft language (drafts rarely enumerate failures). The model produces 2–3 candidates the user multi-selects from. Default candidates by task type:

| Task type | Likely failure modes |
|-----------|----------------------|
| Code generation | Silent runtime errors on unexpected inputs; missing test coverage of edge cases; over-engineered abstractions; insecure handling of external input |
| Data analysis | Biased conclusions from narrow source coverage; missing edge cases / outliers; unsourced claims; misleading visualizations |
| Research / search | Narrow source coverage; uncritical adoption of dominant framing; stale information; hallucinated citations |
| Writing / content | Off-brand voice; factual drift; verbose hedging; off-target audience |
| Agent / workflow | Misreported completion (writes to disk); destructive action on the model's judgment; over-broad scope; cascading errors across steps |
| Orchestrated research | Under-delegation (orchestrator does the work itself); duplicate subagent scope; shallow synthesis; unverified numeric estimates passed between agents |
| Creative | Generic / safe output; off-genre voice; constraint violations |
| Multi-modal | Misread image content; wrong coordinates (4.6-era scaling); excessive image-token cost |
| API / integration | 400 errors from deprecated parameters; schema-violation outputs; mishandled stop reasons (`refusal`, `model_context_window_exceeded`) |
| Multi-model | Inconsistent output shapes across model routes; routing logic that doesn't match each model's strengths |
| Knowledge work (Cowork) | Memory-write inconsistency across sessions; over-broad file edits; missing application-specific format handling |

Confidence is Medium by default (failure modes are inferences from domain, not from draft text) — and may upgrade to High when the draft explicitly names a failure mode it cares about.

## Question 5 — Output constraints

Inferred from draft structure and explicit format cues.

| Signal | Inferred output format | Default confidence |
|--------|------------------------|---------------------|
| Code-shaped draft, language reference | Code artifact (single file, module, package) | High |
| "Write," "draft," "produce a report" | Prose document (markdown, doc) | High |
| JSON / schema references | Structured JSON output | High |
| "Slide deck," "presentation" | Slide deliverable | High |
| Chat-shaped conversational draft | Chat response | High |
| **No format cue at all** | Default to a markdown document; lead Widget with this and a code/JSON alternative | **Low** |

## Conflicting signals — when one draft shows multiple task-type cues

When 2+ task-type signals appear in a single draft (e.g., code-shaped fragments + research/synthesis verbs; agentic verbs + chat-shaped audience), apply this rule:

- Downgrade confidence to Low or Medium for the leading task-type inference.
- Lead Widget Call 1 with the task type closest to the **deliverable verb** in the draft — what the user said they want produced, not what they said they want examined. ("Write a research brief" → writing/content leads; "Research and write a brief" still leads with writing/content because the deliverable is the brief.)
- Make Option 2 in Widget Call 1 the other competing task type.
- Document the conflict in the Phase 0 restatement prose so the user sees the widget is asking for direction, not just confirmation.

## Confidence calibration rules

The confidence ladder for Step 0C inferences is **High / Medium / Low**. Apply these rules consistently across questions:

1. **High = the draft directly supports the inference.** Explicit code in a code-generation draft. A named platform. A verbatim objective. The signal is *in the draft*; the inference is recognition, not extrapolation.

2. **Medium = there's a concrete cue you're partially trusting.** The cue is present but requires interpretation (a stream-of-consciousness draft with code-shaped fragments; a deployment that's implied by the surrounding context but not named). The inference is reasonable but not certain.

3. **Low = the cue is absent, weak, or contradictory.** No concrete signal in the draft for this question; the inference is the model's guess from priors. **A minimal draft (one sentence, no concrete artifact, no platform mention) defaults to Low for every question — not Medium.** Medium implies a cue you're partially trusting; if there's no cue, it's Low.

### Multi-Low-confidence behavior

When **multiple** Phase 1 questions return Low confidence inferences for a single draft, the Widget Call 1 design changes:

- Widen each Low-confidence question to all 4 option slots (the maximum) so the user has more meaningful choices to confirm or reject.
- For the deployment target question specifically, list the platforms in order of which the model thinks is most likely given the strongest weak cue.
- Surface the Low-confidence assessment in the Phase 0 restatement prose so the user knows the widget is asking for direction, not just confirmation.

The instinct to upgrade confidence to "make Phase 1 feel cleaner" is the failure mode this rule prevents. A Low confidence stamp is itself useful signal downstream — Phase 2.5 may rely on it when deciding how aggressively to ask follow-up questions; Phase 4 may rely on it when deciding whether to embed an "investigate further" clause in the optimized prompt.

## Worked example

Draft input: *"Optimize this for an API-deployed Python ETL: extract from Postgres → transform with pandas → load into Snowflake. Want error handling and tests."*

```
INFERRED ANSWERS:
- Q1 (Task type): Code generation | Signals: "Python ETL", concrete library mention (pandas), code-shaped deliverable | Confidence: High
- Q2 (Deployment target): Claude API | Signals: explicit "API-deployed" | Confidence: High
- Q3 (Primary outcome): Production-ready ETL pipeline with tests and error handling | Signals: stated nearly verbatim — "error handling and tests" + ETL flow specified | Confidence: High
- Q4 (Failure modes): Silent data loss on transform errors; pandas memory blow-up on large extracts; missing test coverage of edge cases | Signals: inferred from ETL domain — draft doesn't enumerate failures, but pandas + cross-system data movement implies these | Confidence: Medium
- Q5 (Output constraints): Single .py file or module structure (not chat) | Signals: "Python" implies a code artifact, not a chat response — but format not stated explicitly | Confidence: Medium
```

The user confirms or edits each via Widget Calls 1 and 2. Inferences populate option *order* (lead with the inferred value) — they do not bypass the widget. The widget always fires.

## Cross-references

- See `SKILL.md` Phase 0 Step 0C for the structured output template.
- See `SKILL.md` Phase 1 for how the inferences populate Widget Calls 1, 2, 3a, 3b.
- See [task-heuristics.md](task-heuristics.md) for task-type-specific handling once task type is confirmed.
- See [orchestrated-research.md](orchestrated-research.md) when the sniff rule activates and orchestrated-research becomes the lead option.

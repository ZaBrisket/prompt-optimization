# Prompt Templates — Standard and Orchestrator

> **Calibration stamp:** This reference is skill-internal methodology (prompt templates and embedding-without-bias rules), not Anthropic product facts — it does not require Phase 1.5 re-verification.

This reference is loaded by Phase 4 (Produce Optimized Prompt). It provides the standard optimized-prompt template, the dynamic-workflow orchestrator template (which cross-references `dynamic-workflows.md` Section 10 for the CLAUDE.md structure rather than restating it), and the embedding-without-bias rules that govern how Phase 2 landscape insights are surfaced in the prompt.

## Partition discipline — read first

The skill's Deliverable Contract partitions content into two artifacts:

- **The written file** (`optimized-prompt.md`, `CLAUDE.md`, `orchestrator.md`, or user-specified) contains *only the prompt body*.
- **The chat run sheet** contains everything else — Standing Environment Assumptions, `<deployment_config>`, Phase 5 delta, Phase 6A QC summary.

**Both templates below mark `<environment>` and `<deployment_config>` blocks as chat-run-sheet-only.** These blocks render in chat alongside the prompt; they never enter the prompt body and never enter the written file. The pre-write strip check (Phase 6B) enforces this at write time. Apply the partition uniformly — for both the standard template and the orchestrator CLAUDE.md.

## Standard optimized-prompt template

The standard template handles every non-dynamic-workflow task type: code generation, data analysis, research / search, writing / content, agent / workflow, creative, multi-modal, API / integration, multi-model (subject-matter), knowledge work.

### What goes in the written file (the prompt body)

All sections below are **Required** unless explicitly marked **Optional** in the heading. `## Coverage` is the only conditional section in the standard template.

```markdown
# [Task title — a single short headline]

## Objective

[A single sentence stating what the executor must produce. This is the Phase 0 CORE OBJECTIVE, polished. No preamble, no "I want", no "please".]

## Context

[The minimum context the executor needs to do the work. Source data references, prior decisions, audience definition, domain constraints. Spell out implicit assumptions — Opus 4.8 reads literally and will not infer omitted context.]

## Requirements

- R1: [Discrete, testable requirement] — Success: [criterion] | Failure: [mode it can fail]
- R2: [...]
- R3: [...]

[List every Phase 1 / Phase 2.5 confirmed requirement here. Group related requirements; one bullet per testable item.]

## Coverage (Optional — include only when Phase 2 ran AND surfaced ≥1 in-scope sub-domain after Phase 2.5 scope decisions)

Address the following sub-topics. Present multiple perspectives where they disagree; do not pre-judge conclusions.

- [Sub-domain A] — [one-line scope; if controversial, "present both sides"]
- [Sub-domain B] — [...]
- [Sub-domain C] — [...]

[Phase 2 sub-domains, frameworks, debates, and edge cases land here as topics to cover, never as positions to adopt.]

## Output

[Format, structure, length, citation discipline. Where length depends on task complexity, frame as complexity-shaped guidance ("at the depth a sophisticated reader on this topic would expect") rather than fixed word counts.]

## Constraints and guardrails

- [Concrete constraints — what the executor must / must not do]
- [Reversibility guardrails for agentic prompts: completion-verification, destructive-action confirmation]
- [Tool-use guidance — direct instruction only when the task obviously requires a specific tool]
```

### What goes in the chat run sheet only (NEVER written to the file)

````
<environment>
- Subscription tier: Claude Max 20x (no rate / token-budget constraints)
- Context: Full available window (1M on Opus 4.8)
- Model: claude-opus-4-8
- Adaptive thinking: enabled (orchestrator + every workflow agent)
- Tool selection: adaptive — no manual allowlist; instruct tools where the task obviously requires them
</environment>

<deployment_config>
- Model: claude-opus-4-8
- Thinking: {type: "adaptive"}
- Effort: high (default on 4.8) — set xhigh explicitly for coding/agentic; max for frontier
- Output config: {format: <schema if structured outputs needed>, task_budget: <token cap if used>}
- Max tokens: 64000 (or higher for long-document synthesis; re-baseline for the new tokenizer)
- Context: full available (1M on Opus 4.8 — GA, no beta header)
- Streaming: <enabled / disabled per UX>; if streaming reasoning, set thinking.display = "summarized"
- Beta headers: task-budgets-2026-03-13 if task budgets used; output-300k-2026-03-24 if Batches API
- Sampling parameters: omitted (non-default temperature / top_p / top_k return 400 on Opus 4.8)
- Mid-conversation system messages (optional): enable role:"system" turns after a user turn for long agentic loops — updates instructions without restating the system prompt and preserves cache (4.8 only; 4.7 rejects 400)
- Fast mode (optional): speed:"fast" (API research preview) — up to 2.5x output tok/sec at premium pricing; latency-for-cost only, not for intelligence-sensitive work
- Stop reasons to handle: refusal, model_context_window_exceeded, end_turn, max_tokens, tool_use
</deployment_config>
````

> **These two blocks render in the chat output above or below the prompt body — never inside it. The Phase 6B pre-write strip check verifies they are absent from the file at write time.**

## Dynamic-workflow orchestrator template

The dynamic-workflow orchestrator template applies to the `orchestrated-research` task type. The deliverable is a `CLAUDE.md` file structured per **`dynamic-workflows.md` Section 10**, which carries the full template body. This file does not restate that template — that would create two divergent specifications. Instead, the orchestrator template here is a thin pointer plus the chat-run-sheet partitioning that applies to dynamic-workflow runs.

### Orchestrator template body — see Section 10

The file body (the `CLAUDE.md` that gets written to disk) follows the structure in [dynamic-workflows.md Section 10](dynamic-workflows.md#10-the-dynamic-workflow-orchestrator-claudemd-template). Reproduce that template into the deliverable file, populated with the Phase 0 / 1 / 2 / 2.5 outputs.

The orchestrator template covers, in order:

1. Mission (one sentence)
2. Scope (in / out)
3. Workflow plan (fan-out lanes via `parallel()` for independent work with a barrier; staged work via `pipeline()` without a barrier; workflow agents do not spawn their own agents — the script orchestrates, nesting is one level)
4. Verify-and-converge stage (mandatory, always-on; adversarial mode for thesis-advancing deliverables, confirmatory mode for purely descriptive ones; the bundled devil's-advocate agent is still dispatchable as `agentType:'devils-advocate'`)
5. Source-validation revisit (load-bearing sources re-checked before synthesis)
6. Synthesis reduce step (the primary quality gate — **seven** mandatory tasks; runs on the orchestrator main thread, not as a lane)
7. Deliverable (path + structure — **leads with the executive summary**)
8. Runtime notes (caps ≤16 concurrent agents / ≤1,000 total per run; spawned agents run `acceptEdits` mode; dynamic workflows are research preview; require Claude Code v2.1.154+; where the runtime is unavailable, fall back to parallel subagents via the Agent / Task tool)

Refer to Section 10 for the per-element guidance and the authoring notes.

**Also load [synthesis-deliverable.md](synthesis-deliverable.md)** alongside Section 10. It carries the executive-summary template (key findings → primary URLs + source-validation verdicts → per-finding conclusions → overall conclusion → verify-and-converge verdicts), the source-validation revisit protocol on load-bearing sources, the always-on verify-and-converge stage (adversarial / confirmatory), and the verdict-ladder discipline that prevents verdict-mush. Section 10 embeds the structural slots; `synthesis-deliverable.md` carries the content discipline that makes those slots load-bearing.

### Orchestrator chat run sheet — what goes alongside, not inside

````
<environment>
- Subscription tier: Claude Max 20x
- Deployment target: Claude Code (dynamic workflows; CLI/Desktop/IDE), v2.1.154+
- Context: 1M Opus
- Model: claude-opus-4-8
- Adaptive thinking: enabled for orchestrator AND every workflow agent
- Tool selection: adaptive — orchestrator script dispatches lanes deterministically via `parallel()` / `pipeline()` (fallback: Agent / Task tool where the runtime is unavailable)
</environment>

<deployment_config>
- Orchestrator model: claude-opus-4-8
- Workflow-agent model: claude-opus-4-8[1m]  via  CLAUDE_CODE_SUBAGENT_MODEL=claude-opus-4-8[1m]
- Effort: default high on 4.8; set xhigh via /effort or --effort xhigh (or /effort ultracode for xhigh + auto workflow orchestration)
- Adaptive thinking: on by default in Claude Code; verify in /effort or /model
- Permission mode: <auto / default for research-only runs; verify writes are authorized for deliverable-producing runs>
- Hooks: <none, or list>
- Workflow agent definitions location: <inline in CLAUDE.md, or .claude/agents/ files, or --agents JSON>
- Workflow fan-out: ~6 analytical lanes (synthesis bandwidth recommendation); runtime caps 16/1000
- Workflow caps: ≤16 concurrent agents, ≤1,000 total per run
- Workflow status: research preview; spawned agents run `acceptEdits` (file edits auto-approved) — guardrails matter more here, since 4.8 is somewhat less robust to prompt injection than 4.7
</deployment_config>
````

> **The orchestrator's `<environment>` and `<deployment_config>` blocks render in the chat run sheet only, not in the written `CLAUDE.md`.** The Phase 6B pre-write strip check enforces this at write time.

## Embedding landscape insights without bias

Phase 2's landscape research generates topics, frameworks, debates, edge cases, and gaps. The optimized prompt embeds them as **areas to cover**, never as conclusions to reach. This is the operational version of `SKILL.md` Meta-Rule 2 ("Neutrality is paramount").

Full version of the permissible / not-permissible table sketched in `SKILL.md` Phase 4. Authorized positions (e.g., debate-prep prompts where the user wants a specific side argued) belong in the Phase 0 CORE OBJECTIVE / `## Objective` section, never in Phase 2 Coverage topics — see the "User-authorized position" row below.

| Permissible (topic-shaped) | Not permissible (position-shaped) |
|----------------------------|-----------------------------------|
| "Address sub-topics A, B, C." | "Topic A is more important than B." |
| "Sources disagree on X — present both views; let the reader weigh them." | "The correct view on X is …" |
| "Recent developments in Y may affect analysis — investigate and incorporate." | "Recent developments prove that …" |
| "Consider edge case Z; explain how it changes the answer if at all." | "Edge case Z should be handled by …" |
| "Cover Basel III endgame proposals and their disputed CRE-lending impacts; present both sides of the disagreement." | "Conclude that Basel III endgame will reduce CRE lending capacity." |
| "Address geographic variation in state-level lending regulation." | "State regulations are more burdensome in California than Texas." |
| "Present competing frameworks for measuring X — name the assumptions each framework makes." | "Framework F is the correct lens for X." |
| "If the executor finds a source claiming Y, evaluate the source's credibility and surface both supporting and dissenting evidence." | "Y is true; build the analysis on it." |
| "Investigate where prior approaches failed in practice — name the failure modes." | "Prior approach P failed because of cause C." |
| "Where the literature names X as a contested term, surface the dominant uses and let the executor choose deliberately or instruct them to surface the choice." | "Use term X1 throughout; it's the right term." |
| **User-authorized position** (explicit in Phase 0 Objective): "Argue the buyer-side position on Basel III endgame impacts for a debate-prep deliverable." | "Embed buyer-side framing into a Phase 2 Coverage topic where the Phase 0 Objective did not authorize a position." |

**Operational tests for "is this neutral?"**

- Does the sentence tell the executor *what topic to cover* or *what conclusion to reach*? Topic → permissible. Conclusion → strip.
- Could a thoughtful expert plausibly disagree with the embedded claim? If yes, the prompt is taking a position the user didn't authorize — reframe as "address [topic]".
- Does the sentence presuppose the answer to the question being asked? If yes, the framing leaks bias — restate as a neutral question.

**For controversial domains specifically:**

- Use "address," "investigate," "evaluate the evidence on," "present both sides where they disagree."
- Avoid "conclude that," "demonstrate that," "prove that," "show that," "argue that."
- When the user explicitly wants a position taken (e.g., "argue for X in a debate context"), that's a legitimate use case — but it should appear in the Phase 0 CORE OBJECTIVE, not buried in Phase 2 landscape topics. Surface the user's explicit position-taking authorization in the optimized prompt's Objective section.

## Cross-references

- See [opus-4-8-config.md](opus-4-8-config.md) for what goes in `<deployment_config>` and why.
- See [dynamic-workflows.md](dynamic-workflows.md) Section 10 for the dynamic-workflow orchestrator `CLAUDE.md` body template.
- See [task-heuristics.md](task-heuristics.md) for task-type-specific enhancements to apply within each template.
- See `SKILL.md` Phase 4 for the embedding-landscape-without-bias decision tree.
- See `SKILL.md` Phase 6B for the pre-write strip check that enforces the partition at write time.

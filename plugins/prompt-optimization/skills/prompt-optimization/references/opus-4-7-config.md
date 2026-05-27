# Opus 4.7 Configuration & Critical Behaviors

> **Calibration stamp:** Facts in this reference verified against authoritative Anthropic documentation on **2026-05-20**. Re-verify on every skill run via Phase 1.5.

This reference is loaded by the Opus 4.7 Configuration section of `SKILL.md` and consulted whenever the optimized prompt's `<deployment_config>` block is being constructed. It captures the model's API surface, the changes from Opus 4.6, and the behaviors that should shape a 4.7-targeted prompt.

## Table of contents

1. [Critical Behaviors table](#critical-behaviors-table)
2. [API parameters](#api-parameters)
3. [Effort levels](#effort-levels)
4. [Adaptive thinking](#adaptive-thinking)
5. [Output ceiling and streaming](#output-ceiling-and-streaming)
6. [Structured outputs (no prefilling)](#structured-outputs-no-prefilling)
7. [Context window options](#context-window-options)
8. [Long-context performance and verification posture](#long-context-performance-and-verification-posture)
9. [Sources](#sources)

---

## Critical Behaviors table

The table partitions behaviors by whether they carry forward from Opus 4.6 or are new in 4.7. Both halves matter when writing an optimized prompt — the 4.6-carryforward items are easy to forget because they already shaped Opus 4.6 prompts; the 4.7-new items are easy to miss because they are recent.

### Carries forward from Opus 4.6

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **Prefilling assistant messages returns 400.** | Never instruct prefilling. For format control use structured outputs, system-prompt instructions, or `output_config.format`. |
| **1M-token context window at standard pricing.** | The deployment config can use the full 1M context without a long-context premium; no beta header required. |
| **128K max output tokens** on the synchronous Messages API. | Re-baseline `max_tokens`. The optimized prompt's `<deployment_config>` block in the chat run sheet should specify the output budget the downstream session can use. |
| **Adaptive thinking is the supported reasoning surface.** | The thinking field is `{type: "adaptive"}`. Manual extended thinking with `budget_tokens` is gone. |
| **`output_format` parameter is deprecated** (still functional). | Use `output_config.format` instead. Flag this in Phase 3 anti-patterns. |
| **No `temperature` / `top_p` / `top_k`** when a non-default value is set (400 error on Opus 4.7). | Strip sampling-parameter instructions from drafts; steer behavior through prompting. |

### New in Opus 4.7

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **New `xhigh` effort level.** | The effort ladder is `low / medium / high / xhigh / max` — five levels, not four. `xhigh` is the recommended default for coding and agentic work. |
| **Adaptive thinking is OFF by default on the raw API.** | Requests with no `thinking` field run without thinking. The Standing Environment Assumption requires adaptive thinking, so the chat-run-sheet `<deployment_config>` must explicitly set `thinking: {type: "adaptive"}`. |
| **Thinking display defaults to `omitted`.** | Set `thinking.display: "summarized"` to surface reasoning to users. Without it, downstream UIs may show a long pause before output begins. |
| **New tokenizer — 1×–1.35× more tokens** than Opus 4.6 for the same text. | Re-baseline any token budgets carried over from a 4.6 prompt. Compaction triggers and `max_tokens` need headroom. |
| **Stricter effort calibration.** | At `low`/`medium` the model scopes work tightly. If you observe under-thinking on complex tasks, raise effort to `high` or `xhigh` rather than wrapping the prompt with "think carefully" exhortations. |
| **Fewer subagents spawned by default.** | The orchestrator under-delegates — give explicit dispatch instructions. Meta-Rule 13 exists because of this. |
| **Fewer tool calls by default.** | Reasoning is preferred over tool calls. To raise tool usage, raise effort (`high`/`xhigh`) and add explicit tool-use guidance. |
| **More literal instruction following.** | The model won't silently generalize a rule from one item to another. Spell each item out. Negative examples ("don't do X") work less well than positive examples of the desired pattern. |
| **Response length calibrates to task complexity.** | Replaces 4.6's fixed verbosity baseline. If output is too short or too long, tune via prompt rather than expecting a global verbosity dial. |
| **More direct, opinionated tone.** | Less validation-forward phrasing, fewer emoji. If a downstream product depends on a particular voice, re-evaluate style prompts against the new baseline. |
| **Built-in agentic progress updates.** | The model emits its own status updates during long agentic traces. Remove scaffolding like "summarize progress every 3 tool calls." If updates are mis-calibrated, instruct explicitly and give examples. |
| **High-resolution image support** (up to 2576px long edge, ~3.75MP). | Image tokens can be ~3× higher than on 4.6. Downsample before sending if you don't need the fidelity. |
| **Real-time cybersecurity safeguards.** | Refusals on prohibited/high-risk cyber topics are stricter. For legitimate security work, route through the Cyber Verification Program. |

## API parameters

The minimum-viable Opus 4.7 request shape, sufficient for most optimized prompts targeting the API:

```python
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,                       # Start at 64K when running xhigh/max effort
    thinking={"type": "adaptive"},          # Must be explicit — OFF by default
    output_config={"effort": "xhigh"},      # Or "high" / "max" — see effort ladder
    messages=[{"role": "user", "content": "..."}],
)
```

### Variant request shapes

For common deployment shapes beyond the minimum-viable example above, the canonical request bodies are:

**Research with web search tool:**
```python
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "xhigh"},
    tools=[{"type": "web_search_20250305"}],
    messages=[{"role": "user", "content": "..."}],
)
```

> The web-search tool version above is calibration-stamped — Phase 1.5 Query 5 confirms the current `web_search_*` tool version at runtime before it lands in an optimized prompt.

**Structured JSON output via `output_config.format`:**
```python
client.messages.create(
    model="claude-opus-4-7",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={
        "effort": "high",
        "format": {"type": "json_schema", "json_schema": {...}},
    },
    messages=[{"role": "user", "content": "..."}],
)
```

**Orchestrated-research dispatch (Claude Code CLI):** orchestrator runs under `CLAUDE_CODE_SUBAGENT_MODEL=claude-opus-4-7[1m]` and `CLAUDE_CODE_EFFORT_LEVEL=xhigh`; the `Agent` / `Task` dispatch is built into the CLI. Subagent calls inherit the env-var-set model per the Standing Environment Assumption.

### Parameters to remove from any 4.6-era draft

| Remove | Why |
|--------|-----|
| `temperature`, `top_p`, `top_k` (when non-default) | 400 error on Opus 4.7. |
| `thinking: {type: "enabled", budget_tokens: N}` | Replaced by adaptive thinking + effort. |
| Assistant-message prefills | 400 error on Opus 4.7 (carried over from 4.6). |
| `betas=["effort-2025-11-24"]` | Effort is GA. Header has no effect. |
| `betas=["interleaved-thinking-2025-05-14"]` | Interleaved thinking is automatic in adaptive mode. |
| `betas=["fine-grained-tool-streaming-2025-05-14"]` | GA. |
| `output_format={...}` | Deprecated — use `output_config={"format": {...}}`. Still functional but flagged for future removal. |
| `betas=["token-efficient-tools-2025-02-19"]`, `betas=["output-128k-2025-02-19"]` | Built-in on Claude 4+; legacy headers have no effect. |

### Beta features still active

| Feature | Header |
|---------|--------|
| Task budgets (advisory token cap across the full agentic loop; minimum 20K) | `task-budgets-2026-03-13` |
| 300K output tokens via Batches API | `output-300k-2026-03-24` |

## Effort levels

| Level | When to use |
|-------|-------------|
| `low` | Short, scoped, latency-sensitive work that is not intelligence-sensitive. |
| `medium` | Cost-sensitive work that can trade off some intelligence. |
| `high` | Minimum for intelligence-sensitive work. Balances token usage and capability. |
| `xhigh` | **Claude Code default on Opus 4.7** (Messages API defaults to `high` — set `xhigh` explicitly). Best results for most coding and agentic tasks. |
| `max` | Can deliver gains on demanding tasks but may overthink. Session-only (except via the `CLAUDE_CODE_EFFORT_LEVEL` env var). Test before adopting broadly. |

Effort is calibrated per model. The same level name does not map to the same internal value across Opus 4.6 / Sonnet 4.6 / Opus 4.7. Effort matters more on 4.7 than on any prior Opus — experiment when migrating.

Setting `xhigh` on a model that doesn't support it (e.g., Opus 4.6) silently falls back to that model's highest supported level. Inverse trap: if you observe under-thinking on a complex task at `low`, **raise the effort level rather than prompting around the under-thinking**.

### Decision rule for effort selection

- `xhigh` is the default for coding, agentic, and analytical work. The Standing Environment Assumption (no rate limits, no token-budget constraints) means latency and cost are rarely the binding constraint.
- Drop to `high` only when the deployment has a hard latency budget (< ~5s response, e.g., an interactive UI) AND the work is not intelligence-sensitive enough to require `xhigh`.
- Consider `max` only when (a) you observe under-thinking on a representative draft at `xhigh`, AND (b) the work is intelligence-sensitive enough to justify potential overthinking (max can over-deliberate on simple tasks). Test before adopting broadly.
- Never default to `medium` or `low` for the skill's optimized prompts — they are intelligence-sensitive by construction.

## Adaptive thinking

### Syntax

```python
thinking = {
    "type": "adaptive",
    "display": "summarized",   # default is "omitted" on 4.7
}
output_config = {"effort": "xhigh"}
```

### What changed

- Manual extended thinking (`{type: "enabled", budget_tokens: N}`) returns 400.
- Adaptive thinking is the only supported thinking mode on Opus 4.7.
- Interleaved thinking (reasoning between tool calls) is automatic in adaptive mode. No beta header required.
- Adaptive thinking is OFF by default — requests with no `thinking` field run without thinking, matching Opus 4.6's no-thinking default.

### Steering depth without changing effort

For one-off deeper reasoning at a fixed effort level, include the keyword `ultrathink` anywhere in the prompt — Claude Code recognizes it as an in-context instruction without changing the session effort setting. Other phrases (`think hard`, `think more`) pass through as ordinary prompt text and are not special.

## Output ceiling and streaming

- **Synchronous Messages API:** 128K max output tokens.
- **Message Batches API:** 300K max output tokens with the `output-300k-2026-03-24` beta header.
- Start `max_tokens` at 64K when running `xhigh`/`max` effort. Tune from there.
- The new tokenizer can produce 1×–1.35× more tokens than Opus 4.6 for the same text — re-baseline `max_tokens` and any compaction triggers.

Streaming is supported. If the chat run sheet's deployment context streams reasoning to users, set `thinking.display: "summarized"` so users see visible progress during thinking instead of a long pause.

## Structured outputs (no prefilling)

Prefilling assistant messages returns a 400 error on Opus 4.7. The replacement surface depends on the use case:

| Prior use case for prefill | Replacement |
|----------------------------|-------------|
| Forcing JSON / YAML output | Use `output_config.format` with a JSON schema, or tools with enum fields for classification. Add a system-prompt fallback like "Output ONLY valid JSON with no preamble." |
| Eliminating preamble phrases ("Here is…", "Based on…") | System prompt: "Respond directly without preamble. Do not start with phrases like 'Here is…', 'Based on…', etc." |
| Avoiding bad refusals | Clear user-message prompting is usually sufficient; refusal calibration has improved. |
| Continuation after interruption | Move the continuation into the user message: "Your previous response was interrupted and ended with `[…]`. Continue from there." |
| Context hydration in long conversations | Inject the reminders into the user turn, not the assistant turn. |

For JSON output, the canonical Opus 4.7 shape is:

```python
output_config = {
    "effort": "high",
    "format": {"type": "json_schema", "json_schema": {...}},
}
```

`output_format={...}` (top-level) is deprecated. Migrate to `output_config={"format": {...}}`.

## Context window options

- **Standard context on Opus 4.7: 1M tokens.** GA on the Claude API, at standard input pricing, with no long-context premium.
- **Claude Code:** Max / Team / Enterprise plans get 1M-context Opus automatically. The picker exposes `opus[1m]` or, by full ID, `claude-opus-4-7[1m]`. The `[1m]` suffix can also be applied to `ANTHROPIC_DEFAULT_OPUS_MODEL` for pinned deployments.
- To disable the 1M variant in Claude Code, set `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`.
- The Standing Environment Assumption (full available context window, no cost constraints) means the optimized prompt should not include token-conservation language in its instructions. Where partitioning is needed (long-context synthesis), partition into sub-windows under 256K rather than swapping models.

## Long-context performance and verification posture

Opus 4.7's release positioning emphasizes strong long-context performance. The release announcement (April 16, 2026) and the model card describe 4.7 as state-of-the-art on long-context reasoning. Anthropic's published evaluations on the MRCR (Multi-Round Co-Reference Resolution) family of benchmarks show Opus 4.6 achieving 76% on the 8-needle 1M variant of MRCR v2 — strong but not perfect — and Opus 4.7 has not been positioned as a regression on this dimension.

**Operating posture for the optimized prompt.** No known long-context regression on Opus 4.7 should be embedded as an assumption. The prudent posture is:

- The 1M context is usable; the optimized prompt may freely instruct multi-document analysis, long-document synthesis, large codebase navigation.
- Retrieval reliability at the very long end (hundreds of thousands of tokens, dense distractors, exact-match recall over many similar items) is an evaluation category where users should validate against their own workload before betting business-critical accuracy on it.
- For high-stakes long-context retrieval, the optimized prompt may instruct intermediate verification — e.g., "after extracting Y, re-check the relevant section to confirm" — as a guardrail. This is general best practice, not a 4.7-specific fix.

## Sources

All accessed 2026-05-20:

- [Migrating to Claude Opus 4.7 — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide) — breaking changes, behavioral changes, GA / beta header status, code samples.
- [Models overview — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models) — model IDs, context windows, output limits, knowledge cutoff, pricing.
- [Building with extended thinking — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) — adaptive thinking syntax, interleaved thinking, display defaults.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — effort ladder per model, `[1m]` syntax, env vars, default effort.
- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — `CLAUDE_CODE_SUBAGENT_MODEL`, subagent model resolution order, frontmatter fields.
- [Introducing Claude Opus 4.7](https://www.anthropic.com/news/claude-opus-4-7) — release announcement, April 16, 2026.

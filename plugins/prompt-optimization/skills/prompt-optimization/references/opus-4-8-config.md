# Opus 4.8 Configuration & Critical Behaviors

> **Calibration stamp:** Facts in this reference verified against authoritative Anthropic documentation on **2026-05-28**. Re-verify on every skill run via Phase 1.5.

This reference is loaded by the Opus 4.8 Configuration section of `SKILL.md` and consulted whenever the optimized prompt's `<deployment_config>` block is being constructed. It captures the model's API surface, what carries forward from Opus 4.7, what is new in 4.8, and the behaviors that should shape a 4.8-targeted prompt.

## Table of contents

1. [Critical Behaviors table](#critical-behaviors-table)
2. [API parameters](#api-parameters)
3. [Effort levels](#effort-levels)
4. [Adaptive thinking](#adaptive-thinking)
5. [New in Opus 4.8](#new-in-opus-48)
6. [Output ceiling and streaming](#output-ceiling-and-streaming)
7. [Structured outputs (no prefilling)](#structured-outputs-no-prefilling)
8. [Context window options](#context-window-options)
9. [Long-context, honesty, and safety posture](#long-context-honesty-and-safety-posture)
10. [Sources](#sources)

---

## Critical Behaviors table

The table partitions behaviors by whether they carry forward from Opus 4.7 or are new in 4.8. Both halves matter — the 4.7-carryforward items are easy to forget because they already shaped 4.7 prompts; the 4.8-new items are easy to miss because they are recent. **There are no breaking API changes from Opus 4.7 to Opus 4.8: code that runs on 4.7 runs unchanged on 4.8 after the model-string swap.**

### Carries forward from Opus 4.7

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **Prefilling assistant messages returns 400.** | Never instruct prefilling. For format control use structured outputs, system-prompt instructions, or `output_config.format`. |
| **No `temperature` / `top_p` / `top_k`** when a non-default value is set (400 error). | Strip sampling-parameter instructions from drafts; steer behavior through prompting. |
| **Manual extended thinking (`budget_tokens`) returns 400.** | Adaptive thinking is the only supported thinking mode. |
| **Adaptive thinking is OFF by default on the raw API.** | The chat-run-sheet `<deployment_config>` must explicitly set `thinking: {type: "adaptive"}`. |
| **The `xhigh` effort level exists** (ladder is `low / medium / high / xhigh / max`). | `xhigh` is available on Opus 4.7 and Opus 4.8 only. |
| **1M-token context window; 128K max output tokens** (sync Messages API). | Re-baseline `max_tokens`; no long-context premium, no beta header. |
| **New tokenizer (1×–1.35× more tokens than Opus 4.6).** | Re-baseline any budgets carried from a 4.6-era prompt. |
| **More literal instruction following; positive examples beat negative.** | Spell each item out; don't rely on "don't do X". |
| **Response length calibrates to complexity; direct tone; built-in agentic progress updates.** | Don't scaffold verbosity dials or "summarize every N tool calls". |
| **High-resolution image support (up to 2576px long edge, ~3.75MP; 1:1 bounding-box coordinates).** | Remove any 4.6-era scale-factor conversion logic. |
| **Real-time cybersecurity safeguards.** | For legitimate security work, route through the Cyber Verification Program. |

### New in Opus 4.8

| Behavior | What this means for the optimized prompt |
|----------|-------------------------------------------|
| **Effort default is now `high` on ALL surfaces** (Claude API *and* Claude Code). | The 4.7-era "`xhigh` is the Claude Code default" is **gone**. Set `xhigh` explicitly for coding and agentic work; the default no longer does it for you. |
| **Effort levels recalibrated** — `medium` allows somewhat more thinking, `high` somewhat less, `xhigh` substantially more, vs. 4.7. | Re-baseline any effort level you tuned against 4.7 cost/latency before adjusting it further. |
| **Adaptive thinking decides per-turn whether to think at all.** | Fewer wasted thinking tokens on bimodal workloads at the same effort level. |
| **Better tool triggering** — far fewer cases of skipping a tool call the task required (a known 4.7 issue). | Less need to nag "use the tool"; instruct tools only where the task obviously requires one (Meta-Rule 14 still holds). |
| **Better long-horizon agentic coding** — fewer compactions, better compaction recovery, better long-context quality. | Long unattended agentic runs (and dynamic workflows) stay on task longer. |
| **Mid-conversation system messages** — `role:"system"` accepted immediately after a user turn (4.7 and earlier reject with 400). | Update instructions late in a long run without restating the full system prompt — preserves prompt-cache hits. Surface as a deployment option for long agentic loops. |
| **Fast mode** (`speed: "fast"`, Claude API research preview) — up to 2.5× output-tokens/sec at premium pricing ($10/$50 per MTok). | A latency-for-cost tradeoff for output generation (OTPS, not time-to-first-token). Not for intelligence-sensitive work; the Standing Environment favors quality. |
| **Prompt-cache minimum lowered to 1,024 tokens** (lower than 4.7). | Short system prompts / tool definitions that were uncacheable on 4.7 now create cache entries with no code change. |
| **Refusal `stop_details` now publicly documented** (present since 4.7). | Instruct the user's harness to read the refusal *category*, not just the `refusal` stop reason. |
| **Honesty / calibration gains** — ~4× less likely to let flaws in its own code pass unremarked; ~10× less overconfidence vs. 4.7; first model with a 0% rate of misreporting flawed results in one eval. | Self-review is more trustworthy. This **reframes** the old "4.7 occasionally misreports completion" rationale — the completion-verification guardrail is still prudent, but as defense-in-depth, not because the model is unreliable here. |
| **Somewhat *less* robust to prompt injection than 4.7; more willing to begin a task without scrutinizing its intent** (system card; product-level safeguards close the gap in practice). | **Guardrails matter more, not less** — especially for agentic and dynamic-workflow deliverables, where workflow-spawned agents run in `acceptEdits` mode and auto-approve file edits. |

## API parameters

The minimum-viable Opus 4.8 request shape, sufficient for most optimized prompts targeting the API:

```python
client.messages.create(
    model="claude-opus-4-8",
    max_tokens=64000,                       # Start at 64K when running xhigh/max effort
    thinking={"type": "adaptive"},          # Must be explicit — OFF by default
    output_config={"effort": "xhigh"},      # Default is "high"; set xhigh explicitly for coding/agentic
    messages=[{"role": "user", "content": "..."}],
)
```

### Variant request shapes

**Research with web search tool** — add `tools=[{"type": "web_search_20250305"}]` (Phase 1.5 Query 5 re-confirms the current `web_search_*` version at runtime).

**Structured JSON output** — `output_config={"effort": "high", "format": {"type": "json_schema", "json_schema": {...}}}`.

**Mid-conversation instruction update (new in 4.8)** — append a `{"role": "system", "content": "..."}` message right after a user turn to revise guidance mid-run without restating the system prompt (preserves cache hits on earlier turns).

**Fast mode (new in 4.8, API research preview)** — add `speed: "fast"` for up to 2.5× output-tokens/sec at premium pricing. Latency-for-cost only; omit for intelligence-sensitive work.

**Dynamic-workflow orchestration (Claude Code)** — there is no API parameter; the orchestration is authored as a JavaScript workflow at runtime. See [dynamic-workflows.md](dynamic-workflows.md).

### Parameters to remove from any 4.7-or-earlier draft

| Remove | Why |
|--------|-----|
| `temperature`, `top_p`, `top_k` (non-default) | 400 error on Opus 4.8 (unchanged from 4.7). |
| `thinking: {type: "enabled", budget_tokens: N}` | 400 error — replaced by adaptive thinking + effort. |
| Assistant-message prefills | 400 error. |
| `output_format={...}` (top-level) | Deprecated — use `output_config={"format": {...}}`. |
| `betas=["effort-2025-11-24"]`, `["interleaved-thinking-2025-05-14"]`, `["fine-grained-tool-streaming-2025-05-14"]`, `["token-efficient-tools-2025-02-19"]`, `["output-128k-2025-02-19"]` | All GA. Headers have no effect; they signal stale calibration. |
| Any context-window beta header | 1M is the default on Opus 4.8 (Claude API / Bedrock / Vertex); the header can be removed. |

### Beta features still active

| Feature | Header |
|---------|--------|
| Task budgets (advisory token cap across the agentic loop; minimum 20K) | `task-budgets-2026-03-13` |
| 300K output tokens via Batches API | `output-300k-2026-03-24` |

## Effort levels

| Level | When to use |
|-------|-------------|
| `low` | Short, scoped, latency-sensitive work; also a reasonable default for cheap subagents. |
| `medium` | Balanced approach with moderate token savings. |
| `high` | **The default on all surfaces (API and Claude Code). Equivalent to omitting the parameter.** Complex reasoning, difficult coding, agentic tasks. |
| `xhigh` | **Recommended starting point for coding and agentic work** (and exploratory tool-calling / search). Available on Opus 4.7 and Opus 4.8 only. Expect meaningfully higher token usage than `high`. |
| `max` | Reserve for genuinely frontier problems; can overthink on simpler tasks. Session-only except via `CLAUDE_CODE_EFFORT_LEVEL`. |

**Default changed in 4.8: `high`, not `xhigh`.** On Claude Code, Opus 4.7 defaulted to `xhigh`; Opus 4.8 defaults to `high` on every surface. The skill's optimized prompts are intelligence-sensitive by construction, so the decision rule is:

- **Set `xhigh` explicitly** for coding, agentic, analytical, and orchestration work — the Standing Environment Assumption (no rate limits, no token-budget constraints) means latency/cost is rarely the binding constraint, and `high` no longer reaches `xhigh` on its own.
- Use `high` for non-coding intelligence-sensitive work where `xhigh`'s extra token spend isn't justified.
- Consider `max` only when you observe under-thinking at `xhigh` on a representative draft.
- Never default to `medium`/`low` for the skill's optimized prompts.

When running `xhigh`/`max`, start `max_tokens` at 64K and tune. Effort levels were recalibrated between 4.7 and 4.8 — re-baseline before adjusting a level you tuned on 4.7.

## Adaptive thinking

```python
thinking = {"type": "adaptive", "display": "summarized"}   # display default is "omitted"
output_config = {"effort": "xhigh"}
```

- Manual extended thinking (`budget_tokens`) returns 400 — adaptive is the only mode.
- Adaptive thinking is OFF unless `thinking: {type: "adaptive"}` is set explicitly on the raw API. On Claude Code, Opus 4.7+ always use adaptive reasoning.
- On 4.8, the model decides **per turn** whether to think — fewer wasted tokens on bimodal workloads at the same effort level.
- **`ultrathink`** anywhere in the prompt requests deeper reasoning for that turn without changing session effort (Claude Code in-context keyword). `think hard` / `think more` are ordinary text.

## New in Opus 4.8

- **Mid-conversation system messages.** `role:"system"` immediately after a user turn (subject to placement rules) is accepted on 4.8; 4.7 and earlier return 400. Use the top-level `system` field for from-the-start instructions; use a mid-conversation system message to revise instructions later without restating the system prompt — this preserves prompt-cache hits and reduces input cost on long agentic loops.
- **Fast mode (research preview, Claude API).** `speed: "fast"` yields up to 2.5× output-tokens/sec from the same model at premium pricing ($10/MTok input, $50/MTok output — 3× cheaper than fast mode on prior Opus models). Speed applies to OTPS, not time-to-first-token. Out of scope for the skill's intelligence-sensitive prompts unless a user explicitly trades quality budget for latency.
- **Prompt-cache minimum 1,024 tokens** (lower than 4.7) — short prompts cache automatically, no code change.
- **Refusal `stop_details`** is now documented; an API-targeted prompt's harness should read the refusal category alongside the `refusal` stop reason.

## Output ceiling and streaming

- **Synchronous Messages API:** 128K max output tokens.
- **Message Batches API:** 300K with `output-300k-2026-03-24`.
- Start `max_tokens` at 64K for `xhigh`/`max`; account for the tokenizer (1×–1.35× vs 4.6) in compaction triggers.
- If streaming reasoning to users, set `thinking.display: "summarized"`.

## Structured outputs (no prefilling)

Prefilling returns 400. Replacements: `output_config.format` with a JSON schema for forced JSON; system-prompt direction ("Respond directly without preamble…") for preamble suppression; move continuations into the user turn. `output_format={...}` (top-level) is deprecated — migrate to `output_config={"format": {...}}`.

## Context window options

- **1M tokens by default** on the Claude API, Amazon Bedrock, and Vertex AI — standard pricing, no premium, no beta header. **200K on Microsoft Foundry.**
- **Claude Code:** Max / Team / Enterprise get 1M-context Opus automatically; the picker exposes `opus[1m]` or, by full ID, `claude-opus-4-8[1m]`. The `[1m]` suffix also applies to `ANTHROPIC_DEFAULT_OPUS_MODEL`. Disable with `CLAUDE_CODE_DISABLE_1M_CONTEXT=1`.
- On the Anthropic API, the `opus` alias resolves to **Opus 4.8**. **Opus 4.8 requires Claude Code v2.1.154 or later** (`claude update`).
- The Standing Environment Assumption (full context, no cost constraints) means optimized prompts should not include token-conservation language. Where partitioning is needed, partition into sub-windows under ~256K rather than swapping models.

## Long-context, honesty, and safety posture

- The 1M context is usable; optimized prompts may freely instruct multi-document analysis and large-codebase navigation.
- For high-stakes exact-match recall at the very long end, instruct intermediate verification ("after extracting Y, re-check the section to confirm") as general best practice and have users validate against their own workload.
- **Honesty:** 4.8 is materially more calibrated than 4.7 (≈4× less likely to let its own code flaws pass; ≈10× less overconfidence). The completion-verification guardrail (Meta-Rule 11) remains as defense-in-depth, but its rationale is no longer "the model is unreliable here."
- **Safety nuance:** the 4.8 system card reports it is somewhat *less* robust to prompt injection than 4.7 and more willing to start a task without scrutinizing intent; product-level safeguards close the gap in practice. For agentic and dynamic-workflow deliverables — where workflow-spawned agents run in `acceptEdits` mode — keep reversibility and confirmation guardrails strong (Meta-Rule 11; `task-heuristics.md` agent/workflow rows).

## Sources

All accessed **2026-05-28**:

- [What's new in Claude Opus 4.8 — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/whats-new-claude-4-8) — new features, inherited constraints, effort default, fast mode, mid-conversation system messages, cache minimum.
- [Migrating to Claude Opus 4.8 — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/migration-guide#migrating-from-claude-opus-47) — migration checklist; "no breaking API changes from 4.7".
- [Effort — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/effort) — effort levels, `high` default, per-use-case recommendations, recalibration.
- [Context windows — Claude API Docs](https://platform.claude.com/docs/en/build-with-claude/context-windows) — 1M default, Foundry 200K.
- [Models overview — Claude API Docs](https://platform.claude.com/docs/en/about-claude/models/overview) — model IDs, pricing, Jan 2026 knowledge cutoff.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — aliases, `[1m]` suffix, env vars, default effort `high`, v2.1.154 requirement, ultracode.
- [Introducing Claude Opus 4.8](https://www.anthropic.com/news/claude-opus-4-8) — release announcement (2026-05-28), benchmarks, dynamic workflows.
- [Claude Opus 4.8 System Card](https://www.anthropic.com/claude-opus-4-8-system-card) — honesty/calibration gains and prompt-injection robustness nuance.

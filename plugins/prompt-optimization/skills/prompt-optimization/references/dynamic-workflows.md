# Dynamic Workflows — Complete Operational Guide

> **Calibration stamp:** Product facts in this reference (workflow runtime, caps, triggers, research-preview status, version gate) verified against authoritative Anthropic documentation on **2026-05-28**. The authoring-API surface is grounded in the live Claude Code workflow runtime and cross-checked against the public docs' constraints; Anthropic does not yet publish the JavaScript function reference, so re-verify both on every skill run via Phase 1.5.

This reference is the full operational manual for the `orchestrated-research` task type. On Opus 4.8 the deliverable is **a `CLAUDE.md` that orchestrates a Claude Code *dynamic workflow*** — a JavaScript orchestration script Claude authors and a runtime executes. It covers when dynamic workflows apply, the runtime model, the authoring API, how decomposition maps onto workflow primitives, the verify-and-converge quality stage, synthesis, the caps/cost/safety envelope, the load-bearing deliverable template (**Section 10**), and the failure-mode catalog. `SKILL.md` Phase 4 cites **Section 10** by number — that section's heading and number are load-bearing; do not move them.

## Table of contents

1. [Overview — when dynamic workflows apply](#1-overview--when-dynamic-workflows-apply)
2. [What a dynamic workflow is](#2-what-a-dynamic-workflow-is)
3. [The authoring API](#3-the-authoring-api)
4. [Decomposition → workflow primitives](#4-decomposition--workflow-primitives)
5. [Context isolation](#5-context-isolation)
6. [Output scope calibration](#6-output-scope-calibration)
7. [Synthesis — the primary quality gate](#7-synthesis--the-primary-quality-gate)
8. [The verify-and-converge stage](#8-the-verify-and-converge-stage)
9. [Caps, cost, and runtime envelope](#9-caps-cost-and-runtime-envelope)
10. [The dynamic-workflow orchestrator CLAUDE.md template](#10-the-dynamic-workflow-orchestrator-claudemd-template)
11. [Failure mode catalog](#11-failure-mode-catalog)

---

## 1. Overview — when dynamic workflows apply

A dynamic workflow applies when:

- The task is large enough that a single linear pass would run out of context, miss perspectives, or produce a thin synthesis from one vantage.
- The work decomposes into independent (or staged) investigations whose results combine into one deliverable.
- The deployment target is **Claude Code** (CLI, Desktop app, IDE extensions, `claude -p`, or the Agent SDK). Dynamic workflows are a Claude Code capability — see `platform-baseline.md`.

It does not apply when the task is small enough to do in one pass, is genuinely sequential with no parallel branches, or targets claude.ai chat (no workflow runtime).

### What Opus 4.8 + dynamic workflows change vs. the 4.7 subagent model

The previous design rested on a behavioral premise: *"Opus under-delegates by default, so the orchestrator `CLAUDE.md` must explicitly instruct subagent dispatch via the `Agent`/`Task` tool, and the model decides turn-by-turn whether to spawn."* **Dynamic workflows make that premise obsolete.**

1. **The script decides what runs next — not the model, turn by turn.** Claude writes a JavaScript orchestration script; a runtime executes it deterministically. Fan-out is no longer something you coax the model into; it is `parallel([...])` in code. The "orchestrator under-delegation" failure mode is solved *structurally*, not by prompting harder.
2. **Intermediate results live in script variables, not the model's context.** The runtime is isolated from the conversation; only the final answer returns to the session. Context isolation is automatic.
3. **The quality idiom is built into the script.** Independent agents address a problem from different angles, other agents try to refute the findings, and the run iterates until answers converge — the verify-and-converge pattern (Section 8).

When the Phase 0 Step 0C sniff (`inference-heuristics.md`) surfaces orchestration signals, Phase 1's Widget Call 1 leads with `orchestrated-research` as the task type — the user confirms before Phase 2.

## 2. What a dynamic workflow is

A dynamic workflow is **a JavaScript script that orchestrates subagents at scale.** Claude writes the script for the task you describe; a runtime executes it in the background, in an isolated environment, while your session stays responsive.

| Property | Value |
|----------|-------|
| **Status / gate** | Research preview. **Claude Code v2.1.154 or later.** All paid plans (Pro must enable it in `/config`) + Anthropic API access, Amazon Bedrock, Google Cloud Vertex AI, Microsoft Foundry. |
| **Surfaces** | CLI, Desktop app, IDE extensions, non-interactive `claude -p`, Agent SDK. |
| **Concurrency cap** | Up to **16 concurrent agents** (fewer on machines with limited CPU cores). |
| **Total cap** | **1,000 agents per run** (runaway-loop backstop). |
| **Script sandbox** | The script itself has **no direct filesystem or shell access** — only the agents it spawns read, write, and run commands. |
| **Spawned-agent permissions** | Agents always run in **`acceptEdits`** mode and inherit the session's tool allowlist, regardless of the session's permission mode. File edits are auto-approved; non-allowlisted shell/web/MCP calls can still prompt mid-run. |
| **User input** | **No mid-run user input.** Only agent permission prompts can pause a run. For human sign-off between stages, split into separate workflows. |
| **Model** | Every agent uses the session model unless the script routes a stage elsewhere (per-agent `model` override). |
| **Resume / rerun** | Resumable within the same session (completed agents return cached results; the rest run live). Save a run's script as a `/<name>` command in `.claude/workflows/` (project) or `~/.claude/workflows/` (user). |
| **Triggers** | the literal word **`workflow`** in a prompt; **`/effort ultracode`** (xhigh + automatic workflow orchestration per substantive task); a saved/bundled command (e.g. `/deep-research`). |
| **Disable** | `/config` toggle, `"disableWorkflows": true` in settings, or `CLAUDE_CODE_DISABLE_WORKFLOWS=1`. |

The bundled **`/deep-research`** workflow is the canonical reference implementation of the quality idiom: it fans web searches across several angles, cross-checks the sources, **votes on each claim, and drops claims that didn't survive cross-checking** before returning a cited report.

## 3. The authoring API

A workflow script begins with a pure-literal `meta` block and then uses a small set of orchestration primitives. (Anthropic does not yet publish this grammar; it is grounded in the live runtime and the public constraints in Section 2. Treat names as current-as-of the calibration stamp and re-verify via Phase 1.5.)

- **`export const meta = { name, description, phases }`** — required header. `phases` lists `{ title, detail }` entries matching the `phase()` calls. Pure literal — no variables or interpolation.
- **`agent(prompt, opts?)`** — spawn one subagent. `opts`: `label`, `phase`, `schema` (a JSON Schema — the agent is forced to return a validated object), `model` (per-stage model override), `agentType` (dispatch a named custom agent such as the plugin's `landscape-research` or `devils-advocate`), `isolation: 'worktree'` (only when agents mutate files in parallel and would conflict). Returns the agent's text, or the validated object when `schema` is set.
- **`parallel(thunks)`** — run tasks concurrently and **wait for all** (a barrier). A failed thunk resolves to `null`; `.filter(Boolean)` before use. Use only when you genuinely need all results together (e.g. dedup/merge across the full set).
- **`pipeline(items, ...stages)`** — run each item through all stages independently with **no barrier between stages** (item A can be in stage 3 while item B is in stage 1). This is the default for multi-stage work.
- **`phase(title)`** — start a progress group; **`log(message)`** — emit a narrator line.
- **`budget`** — `{ total, spent(), remaining() }` for budget-bounded loops; **`workflow(name, args)`** — run another saved workflow inline (nests one level only); **`args`** — the value passed in at launch.

Constraints the script must respect: ≤16 concurrent / 1,000 total agents; no FS/shell from the script; standard JS built-ins only (no `Date.now()` / `Math.random()` — they break resume; vary by index instead).

## 4. Decomposition → workflow primitives

The 4.7 decomposition axes still apply — they now map cleanly onto primitives instead of prose dispatch instructions:

| Decomposition axis | Workflow expression |
|---|---|
| **Sub-domain / source-type / perspective / time-horizon** lanes, all independent | one `agent(...)` per lane inside `parallel([...])` |
| **Wave-based** (Wave 2 needs Wave 1 results) | `pipeline(items, stage1, stage2, …)` — each item flows through stages without a cross-item barrier |
| **Hybrid (shared constants → parallel)** | compute the constant first (one `agent` or plain code), then `parallel([...])` with the constant interpolated into each prompt |
| **Remediation / unknown-size discovery** | a `budget`-bounded or loop-until-dry `while` loop that keeps spawning finders until K rounds return nothing new |

**Default to `pipeline()`.** Use a `parallel()` barrier only when a downstream stage genuinely needs *all* prior results at once (dedup, merge, count-zero early-exit). Carve lanes by sub-domain / source-type / perspective — never by surface convenience — and give each lane an explicit "not in scope" so lanes don't overlap. Cap fan-out at the **synthesis bandwidth** (≈6 lanes for analytical work is a useful heuristic; the hard runtime cap is 16 concurrent / 1,000 total, so narrow-scope extraction can fan far wider).

## 5. Context isolation

Each agent runs in its own context with its own prompt and tool surface; the script's variables hold the returned summaries, and only the final answer reaches the conversation. This is automatic — you do not have to engineer it. Consequences:

- Everything an agent needs goes into its `agent(prompt)` — there is no live streaming into a running agent.
- Constants every agent must share (a definition, a baseline, a glossary) are interpolated into each agent's prompt, or computed once and passed in.
- Agents return the synthesis-ready summary, not their working notes. Use `schema` to force a structured return.

## 6. Output scope calibration

- **Match synthesis bandwidth.** Long reports from many lanes don't synthesize coherently; use `schema` to bound each agent's return to the fields the reduce step needs.
- **Specify structure, not just length** — a JSON Schema is the strongest form of this.
- **Demand citations.** Every factual claim an agent emits carries a resolvable URL.
- **Allow refusal-shaped output.** "If you cannot close your lane in budget, return what you have and list what remains" — agents that bluff completeness corrupt synthesis.
- **Isolate the deliverable with `<result>` tags.** Anthropic's deep-research harness instructs the model to enclose its final report in `<result>` tags and grades only that span, isolating the deliverable from the intermediate tool transcript. Wrapping the synthesis reduce step's final artifact (or any lane returning a long artifact) in `<result>` tags keeps it cleanly separable from working notes. ([opus-4-8-system-card.md](opus-4-8-system-card.md) §9.)

## 7. Synthesis — the primary quality gate

**Synthesis is the most important part of the workflow.** It is integration, not summary: where lanes agree (and why), where they disagree (and what that means), what they all missed, and what the *combined* findings imply that no single lane could surface. In a workflow this is the **final reduce step** — the orchestrator's main thread (or a dedicated final `agent`) folds the lane returns into the deliverable. Mandatory synthesis tasks:

1. **Cross-lane consistency** — surface every disagreement explicitly; never silently pick one.
2. **Coverage check** — every promised sub-domain is covered or flagged as a gap.
3. **Source-diversity check** — across the full corpus, ≥3 of (primary / academic / practitioner / mainstream / dissenting) are represented.
4. **Bias check** — strip embedded conclusions from lane outputs; re-present as topics-with-evidence.
5. **Synthesized findings** — what the combined work implies. The deliverable's value-add.
6. **Source-validation revisit pass** on load-bearing sources — re-fetch every URL cited as primary support for an executive-summary key finding; verify the attribution; verdicts `verified` / `partially verified — see note` / `not verified` / `unreachable`. See [synthesis-deliverable.md](synthesis-deliverable.md).
7. **Verify-and-converge integration** — fold the Section 8 adversarial verdicts into the executive summary under the verdict-ladder discipline.

Synthesis stays with the orchestrator's reduce step — it is not delegated to an isolated lane (whose output would itself need synthesizing). The Section 8 verify stage is the one sanctioned exception: it is an adversarial *check* that returns evidence and tentative verdicts, not a delegation of synthesis.

## 8. The verify-and-converge stage

This stage replaces the 4.7 "dispatch a `devils-advocate` worker" step with the workflow-native idiom: **independent agents adversarially review the findings before they are reported, and the run iterates until the answers converge.** It is always-on.

- **Adversarial mode** (default for thesis-advancing deliverables — analytical, decision-support, comparative, recommendation): for each key finding, spawn independent agents that search for the strongest sourced evidence the *opposite* is true. Aggregate per-finding verdicts by vote: `confirmed` / `refuted by adversarial pass` / `unresolved — both views credible`. The bundled `/deep-research` workflow is the reference shape (fan-out → cross-check → vote → drop non-surviving claims).
- **Confirmatory mode** (for purely descriptive deliverables — inventory, fact-extract, roster): audit each included entry against the inclusion criteria (`included correctly` / `included in error` / `boundary case`) and scout for missed entries.
- **Verdict-ladder discipline (anti-mush guard):** `unresolved` fires only when the adversarial evidence is a sourced, credible counterweight that would survive a Tier-2+ citation in the source-trust hierarchy (`calibration-protocol.md`). Default to `confirmed` when nothing meets the bar. A deliverable where every finding is `unresolved` has drawn no conclusion — that is the failure mode this guards against.

The plugin's bundled `devils-advocate` and `landscape-research` agents (`agents/`) are dispatchable as `agentType` from the workflow script when richer role discipline is wanted; otherwise the equivalent brief is inlined into the verify-stage `agent` prompts. Either way the verdicts feed the synthesis reduce step (Section 7), which owns the final verdict that lands in the deliverable.

## 9. Caps, cost, and runtime envelope

These bound any script Claude writes and must be surfaced in the chat run sheet's `<deployment_config>`:

- **Concurrency ≤16** (fewer on low-CPU machines); **≤1,000 agents total per run.**
- **No mid-run user input** — for human sign-off between stages, structure as separate workflows.
- **The script has no FS/shell access** — only spawned agents touch files or run commands.
- **Spawned agents run `acceptEdits`** (file edits auto-approved). Combined with the 4.8 system card's prompt-injection nuance (`opus-4-8-config.md`), this makes reversibility/confirmation guardrails *more* important on write-capable workflows, not less.
- **Subagent fork-one-directive.** System card §2.3.3 documents that dispatched subagents may exit after a single directive even when they claim to "poll in the background." Express polling or watching as a `budget`-bounded / condition-bounded loop in the script that spawns a fresh agent per iteration — never as a promise a lane keeps running on its own (see §11; [opus-4-8-system-card.md](opus-4-8-system-card.md) §3).
- **Long-run context compaction.** Anthropic's long-running agentic harnesses trigger context compaction around 100K–200K tokens. For long pipeline stages or deep agentic lanes, validate a compaction trigger against the target workload rather than assuming the default ([opus-4-8-system-card.md](opus-4-8-system-card.md) §9).
- **Model:** agents use the session model (`claude-opus-4-8`) unless a stage is routed elsewhere; effort default is `high` — set `xhigh` (or run `ultracode`) for coding/agentic depth.
- **Cost:** a workflow spawns many agents, so a single run uses meaningfully more tokens than the equivalent conversation and counts toward usage and rate limits. The Standing Environment Assumption (no token-budget constraints) authorizes this; still note it for the user.
- **Resume:** within the same session only — exiting Claude Code mid-run restarts the workflow fresh.

## 10. The dynamic-workflow orchestrator CLAUDE.md template

**This section is the load-bearing one — `SKILL.md` Phase 4 cites `dynamic-workflows.md Section 10` by number. Keep this heading and section number stable.**

The deliverable the skill produces is a **`CLAUDE.md` that, when present in a Claude Code session and run with the word `workflow` (or under `/effort ultracode`), causes Claude to author and execute the matching dynamic workflow.** The file does not contain hand-written JavaScript; it is a *declarative orchestration spec* precise enough that Claude writes the correct script from it. The `prompt-template.md` orchestrator template references this section rather than restating it.

### Structure

```markdown
# [Orchestrator name — e.g., "Competitive-landscape diligence brief on [target]"]

> Run this in Claude Code (v2.1.154+) as a dynamic workflow: open the session and
> ask Claude to run the workflow described below (the word "workflow" triggers it),
> or set /effort ultracode. Claude will author the orchestration script and execute it.

## Mission

[A single sentence stating the deliverable the workflow must produce. No preamble.]

## Scope

[In-scope and out-of-scope bullets. The out-of-scope list suppresses gap-flags on excluded sub-domains.]

## Workflow plan

Author a dynamic workflow with these phases. Fan out independent lanes with `parallel`;
sequence dependent stages with `pipeline`. Cap fan-out at the synthesis bandwidth
(~6 analytical lanes; runtime hard caps are 16 concurrent / 1,000 total).

- **Phase 1 — Fan-out research lanes (`parallel`).** One agent per lane below. Each lane carries:
  its precise scope, an explicit "not in scope" (the other lanes), its query taxonomy and
  source-type diversity floor (≥3 of 5), a per-lane search budget, the topic-not-position
  discipline verbatim, and a `schema` that returns key findings + evidence + sources + gaps.
  - Lane A — [scope] / not in scope: [...] / output schema: [...]
  - Lane B — [...]
  - Lane C — [...]
  - [Continue to the synthesis bandwidth.]
- **Phase 2 — Verify-and-converge (always-on).** For each drafted key finding, spawn independent
  adversarial agents (or dispatch `agentType: 'devils-advocate'`) that search for the strongest
  sourced evidence the opposite is true; vote each finding to `confirmed` / `refuted` /
  `unresolved` under the verdict-ladder discipline (`unresolved` needs a Tier-2+ counterweight;
  default `confirmed`). For purely descriptive deliverables, run confirmatory mode instead
  (entry audit + missed-entries scout).
- **Phase 3 — Source-validation revisit.** Re-fetch every load-bearing URL and verify the
  attribution (`verified` / `partially verified` / `not verified` / `unreachable`).

## Shared constants (if applicable)

[Definitions / baselines / glossaries every lane receives in its prompt, with their primary source.
Never pass an unverified number as fact — source it or instruct independent verification.]

## Synthesis (the reduce step — orchestrator main thread, not a lane)

Fold the lane returns into the deliverable. Run the seven synthesis tasks
(consistency, coverage, source-diversity, bias-strip, synthesized findings, source-validation,
verify-and-converge integration). See dynamic-workflows.md §7 and synthesis-deliverable.md.

## Deliverable

Write the final deliverable to [path]. It **leads with the executive summary**
(framing → key findings, each with primary-source URL + validation verdict + confidence →
per-finding conclusions → overall conclusion → verify-and-converge verdicts), then synthesized
findings, supporting evidence by sub-domain, gaps/deferred items, and sources.
Every claim carries a URL; integrative reasoning is explicitly framed ("Combining findings 1 and 3, …").

## Runtime notes (chat run sheet, not the deliverable file)

Caps 16 concurrent / 1,000 total; agents run acceptEdits; no mid-run input; session model
claude-opus-4-8, effort high (set xhigh / ultracode for depth); research preview, Claude Code v2.1.154+.
```

### Authoring notes

- Every lane restates "not in scope" — this differentiates lanes and prevents overlap.
- The deliverable partition mirrors the parent skill's Deliverable Contract: the **orchestration spec, lane prompts, verify-stage findings, and source-validation log are working machinery** (chat run sheet) — never written into the deliverable body. In a workflow these intermediate results live in script variables automatically; the Phase 6B pre-write strip check covers them.
- The executive summary is the deliverable's first section, no exceptions (`synthesis-deliverable.md` § Executive summary template).
- Never pass an unverified numeric estimate as a shared constant — source it or instruct each lane to verify independently.

### Section 10 — Worked example

```markdown
# Competitive-landscape diligence brief on Acme SaaS

> Run in Claude Code (v2.1.154+) as a dynamic workflow (say "run this workflow" or use /effort ultracode).

## Mission
Produce a 5–7 page diligence brief on Acme SaaS's competitive position in mid-market workforce-management software, for a sponsor's IC.

## Scope
- In: direct competitors ($20M–$200M ARR, North America); customer-segment overlap; funding/M&A (18 mo); pricing models.
- Out: adjacent HCM suites; geographies outside North America.

## Workflow plan
- Phase 1 — Fan-out lanes (parallel), each returning {keyFindings, evidence, sources, gaps} via schema:
  - Lane A — direct-competitor inventory (8–12 names). Not in scope: overlap, pricing.
  - Lane B — customer-segment overlap & switching costs. Not in scope: profiles, pricing.
  - Lane C — funding/M&A in the last 18 months. Not in scope: pre-2024-12 history.
  - Lane D — pricing-model survey. Not in scope: discount tactics.
- Phase 2 — Verify-and-converge (adversarial; this brief advances a recommendation): for each key
  finding, independent agents search for the strongest contrary sourced evidence; vote confirmed/
  refuted/unresolved under the verdict-ladder (default confirmed).
- Phase 3 — Source-validation revisit on every load-bearing URL.

## Synthesis
Reduce step runs the seven synthesis tasks; build the executive-summary-led brief.

## Deliverable
Write to diligence-brief-acme.md, leading with the executive summary; every claim carries a URL.

## Runtime notes (chat run sheet)
16/1,000 caps; acceptEdits agents; claude-opus-4-8 @ xhigh; research preview, v2.1.154+.
```

## 11. Failure mode catalog

| Failure | Signal | Fix |
|---------|--------|-----|
| **Script does the work inline** | One long agent; no `parallel`/`pipeline` fan-out in the script | The script must fan out — that is the point of a workflow. Express lanes as `parallel`/`pipeline` agents. |
| **Duplicate lane scope** | Two lanes return overlapping findings | Tighten each lane's "not in scope"; carve by sub-domain/source-type/perspective. |
| **Boilerplate-cloned lane prompts** | Every lane has the same blindspots | Vary scope, source discipline, and `schema` per lane, not just the topic. |
| **Barrier where a pipeline would do** | `parallel` used between stages that don't need all prior results together | Use `pipeline`; reserve `parallel` for genuine dedup/merge/count-zero barriers. |
| **Ignoring the caps** | Script fans out >16 concurrent expecting all to run at once, or loops past 1,000 | Excess concurrency queues (fine); but design to the 16/1,000 envelope and `log()` anything dropped. |
| **FS access from the script** | Script tries to read/write files directly | Only agents touch the filesystem; the script coordinates returns. |
| **Synthesis delegated to a lane** | A "synthesis lane" appears in the fan-out | Synthesis is the reduce step in the orchestrator's main thread. |
| **No verify-and-converge stage** | Findings reported without adversarial review | Phase 2 is always-on (adversarial or confirmatory). |
| **Verdict-mush** | Every finding flipped to `unresolved`; conclusion hedges into uselessness | Apply the verdict-ladder: `unresolved` needs a Tier-2+ sourced counterweight; default `confirmed`. |
| **Missing executive summary** | Deliverable opens with evidence, no key-findings + URL + verdict block | Executive summary is the first section, no exceptions (`synthesis-deliverable.md`). |
| **Source-validation skipped** | Key findings carry no `verified` / `not verified` verdict | Re-fetch every load-bearing URL; never trust the lane's paraphrase. |
| **Working machinery in the deliverable** | Lane prompts / verify findings / source-validation log appear in the written file | Partition: intermediate results stay in script variables / chat run sheet; the Phase 6B strip check enforces it. |
| **Embedded conclusions** | Deliverable states positions rather than topics-with-evidence | Phase 2 neutrality discipline; reframe as "address X; present both sides". |
| **Write-capable workflow with weak guardrails** | Agents run `acceptEdits`; destructive actions auto-approved | On 4.8 (prompt-injection nuance), require explicit confirmation for destructive/irreversible actions and completion-verification on writes. |
| **Lane promises background polling** | A lane prompt says "poll continuously," "keep monitoring," or "run until the condition flips" without a workflow-level loop | §2.3.3 subagent fork-one-directive — express polling as a `budget`-/condition-bounded loop in the script; instruct one-shot lanes to "return when scope is closed or budget is spent; do not wait for further input." |
| **Subagent caveat dropped in synthesis** | A lane reports "could not verify; best guess X" and the reduce step relays X as confirmed | §2.3.3 dropped-caveat fabrication — the reduce step preserves lane uncertainty verbatim and treats "could not verify" as a negative result, not a fact to relay (`synthesis-deliverable.md`). |

## Sources

Accessed **2026-05-28**:

- [Orchestrate subagents at scale with dynamic workflows — Claude Code Docs](https://code.claude.com/docs/en/workflows) — runtime model, caps (16 concurrent / 1,000 total), triggers, research-preview status, v2.1.154 gate, `/deep-research`, disable controls.
- [Introducing dynamic workflows in Claude Code](https://claude.com/blog/introducing-dynamic-workflows-in-claude-code) — design philosophy, verify-and-converge idiom, use cases (codebase audits, large migrations, cross-checked research).
- [Create custom subagents — Claude Code Docs](https://code.claude.com/docs/en/sub-agents) — `agentType` dispatch, subagent definition surface, the one-level nesting constraint.
- [Model configuration — Claude Code Docs](https://code.claude.com/docs/en/model-config) — `ultracode`, effort default `high`, `CLAUDE_CODE_SUBAGENT_MODEL`, v2.1.154.
- Live Claude Code workflow runtime (this session) — the `meta` / `agent` / `parallel` / `pipeline` / `phase` / `budget` authoring surface, cross-checked against the public constraints above.
- [Claude Opus 4.8 System Card](https://www.anthropic.com/claude-opus-4-8-system-card) — §2.3.3 subagent fork-one-directive and dropped-caveat fabrication (the background-polling and caveat-loss failure modes); §8 multi-agent-harness validation, `<result>`-tag deliverable isolation, and long-run context compaction. Routed through [opus-4-8-system-card.md](opus-4-8-system-card.md).
- `SKILL.md` Meta-Rule 13 — the orchestrator invariants this reference operationalizes.

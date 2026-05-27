# Phase 1.5 Calibration Protocol

> **Calibration stamp:** Source-currency stance verified against authoritative Anthropic documentation on **2026-05-20**. The query templates here use `[current year]` / `[current month]` as runtime placeholders — substitute the live session date when running them.

This reference is the operational manual for Phase 1.5. `SKILL.md` Step 1.5B specifies four mandatory queries plus a conditional Query 5; this file expands each into a runnable form, defines the source trust hierarchy that arbitrates conflicts, specifies the delta artifact's structure, gates integration into Phase 4, and prescribes the fallback chain when research fails.

## Why this phase exists

Anthropic ships model updates, new API features, and revised documentation on a rolling basis. The Opus 4.7 behavioral baseline elsewhere in this skill is a snapshot — it can be stale within weeks of any release. Phase 1.5 is the skill's defense against producing a prompt that reflects last month's API. Skipping it because the baseline "looks current" is the failure mode this phase exists to prevent.

## Year-Token Runtime Substitution

Wherever this file shows `[current year]`, `[current month]`, or `[current date]`, substitute the session's real current date at query-construction time. Never hardcode a year. The skill is designed to run for years; a baked-in `2026` will silently mis-scope searches once the calendar rolls.

## Step 1.5B — The query set

### Query 1 (mandatory) — Platform-specific recent guidance

```
Claude Opus 4.7 [deployment target] best practices [current year]
```

Substitute `[deployment target]` with the value confirmed in Phase 1: `API`, `Claude Code CLI`, `Cowork (Desktop)`, or `claude.ai (chat)`. Purpose: surface the most recent platform-scoped guidance, including any prompting changes Anthropic has published since the skill was last calibrated.

### Query 2 (mandatory) — Task-type and feature guidance

```
Anthropic prompt engineering [task type] [feature mentioned in draft]
```

`[task type]` is the Phase 1 selection (code generation, orchestrated research, analysis, etc.). `[feature mentioned in draft]` is the most prominent feature the draft already names — e.g., `prompt caching`, `MCP`, `Files API`, `Computer use`, `Skills`. If the draft names no feature, drop the feature term and search on task type alone. Purpose: pull guidance specific to the work this prompt is actually going to do.

### Query 3 (mandatory) — Deployment-target configuration

```
Claude Opus 4.7 [deployment target] configuration
```

Purpose: confirm the deployment target's setup specifics — required headers, env vars (`CLAUDE_CODE_SUBAGENT_MODEL` for orchestrated research, `ANTHROPIC_DEFAULT_OPUS_MODEL` for pinned deployments), permission modes, output paths, file-write defaults.

### Query 4 (mandatory) — Model-card capabilities + flagship currency

```
Anthropic Claude flagship Opus model card [current year]
```

Purpose: catch capability drift (context window, output ceiling, knowledge cutoff, tokenizer changes, newly deprecated parameters) AND confirm whether the model identifier named in the skill (`claude-opus-4-7` at the calibration-stamp date) is still the current flagship. The model card is the canonical source for the Critical Behaviors table in `opus-4-7-config.md`. If the search surfaces a *newer* flagship model card (e.g., a successor to Opus 4.7), file the finding as a **NEW / UPDATED PRACTICES** delta item with Confidence: High and the action "model-version substitution required per `SKILL.md` § Model-Version Runtime Substitution — apply the new flagship across the optimized prompt's deployment configuration." If the named flagship is still current, file the result under **NO-CHANGE CONFIRMATIONS** attesting that the runtime model identifier is current — no substitution fires.

### Query 5 (conditional) — Surface-area sweep

Fire Query 5 when **any** of:

- (a) the draft references a feature **not** surfaced by Queries 1–4 (e.g., MCP, Files API, Batch API, custom tools, prompt caching, Skills, Cowork, Managed Agents, task budgets);
- (b) the baseline appears stale — Queries 1–4 cite docs more than **30 days** older than the session date;
- (c) Queries 1–4 surface a model retirement notice or deprecation announcement.

Form:

```
Anthropic [specific feature] [deployment target] [current year]
```

Multiple Query 5 invocations are allowed when more than one trigger fires.

### Scoping rules for all queries

- Restrict the search to authoritative Anthropic sources: `docs.anthropic.com`, `docs.claude.com`, `code.claude.com`, `platform.claude.com`, `www.anthropic.com`, system cards, official model cards, official platform guides.
- For research-grade content where third-party signal is helpful (independent benchmarks, ecosystem behavior), surface it but mark it with lower trust per the hierarchy below.
- At least one of Queries 1–5 must explicitly target features unique to the confirmed deployment target. The delta must then surface **two or more platform-unique features**.

## Step 1.5B — Source trust hierarchy

When two sources conflict, the higher-trust source wins. Always cite the trust tier alongside the claim in the delta artifact.

| Tier | Source class | Examples |
|------|--------------|----------|
| **1 — Canonical** | Anthropic-published primary references | Model card, official API docs (`docs.anthropic.com`, `docs.claude.com`), migration guides, system cards |
| **2 — Anthropic supplementary** | Anthropic-authored secondary materials | Engineering blog (`anthropic.com/engineering`), release announcements (`anthropic.com/news`), official cookbooks |
| **3 — Anthropic community / staff** | Anthropic-employee statements in non-doc channels | GitHub issue comments by Anthropic staff, Discord/Slack moderator answers, Anthropic-attributed conference talks |
| **4 — Reputable independent** | Independent measurements and reports | Third-party benchmark papers, established AI research outlets |
| **5 — General web** | Anything else | Forum posts, blog speculation |

**Tie-breaker rule.** When Tier 1 and Tier 2 conflict on a behavioral claim, prefer the **most recent** Tier-1 source. If the most-recent Tier-1 source contradicts an older Tier-1 source, treat the newer one as authoritative and flag the change in the delta.

## Step 1.5C — Current Practices Delta

The delta is the single Phase 1.5 output artifact. It feeds Phase 4 directly and surfaces in the chat run sheet at delivery. Structure it as four named buckets — never a free-form narrative.

```
PHASE 1.5 CURRENT PRACTICES DELTA — accessed [session date]

NEW / UPDATED PRACTICES
- [Practice name] — [one-sentence summary]
  - Source: [URL] (Tier [N])
  - Confidence: [High / Medium / Low]
  - How this affects the optimized prompt: [scope-changing / content-changing / stylistic / no-op]

PLATFORM-SPECIFIC CONSIDERATIONS
- [Feature or constraint unique to the confirmed deployment target]
  - Source: [URL] (Tier [N])
  - Confidence: [High / Medium / Low]
  - How this affects the optimized prompt: [...]

DEPRECATED / SUPERSEDED PATTERNS
- [Pattern] — [what replaces it]
  - Source: [URL] (Tier [N])
  - Confidence: [High / Medium / Low]
  - Flag for Phase 3 anti-pattern sweep: [yes / no]

NO-CHANGE CONFIRMATIONS
- [Baseline claim from this skill that the calibration sweep verified is still current]
  - Source: [URL] (Tier [N])
  - Confidence: [High / Medium / Low]
```

### Authoring rules

- **Cite every claim.** A delta item without a source URL and tier is invalid — drop it or chase the source.
- **Stamp confidence on every item.** A claim derived from a single Tier-3 forum post is not the same as a model-card statement.
- **Never invent a delta.** If the sweep surfaces no changes, the "No-change confirmations" bucket carries the substance and the others may be empty. Confirming baseline is itself a valuable result; fabricating change to make the delta look productive is a worse failure than reporting a clean sweep.
- **Always surface two or more platform-unique features** in the Platform-Specific bucket. If the sweep didn't yield two, run Query 5 against the deployment target until you have two; if you genuinely cannot, document why in a Confidence: Low entry.

### Confidence calibration

| Confidence | Definition |
|------------|------------|
| **High** | Tier-1 source, current (≤30 days old at session date), explicit on the claim. |
| **Medium** | Tier-1 source older than 30 days but still authoritative; or Tier-2 source corroborated by behavioral evidence; or claim involves a slight inferential step. |
| **Low** | Tier-3+ source; or Tier-1 source but the claim requires interpretation; or sources disagree and the tie-break is judgment. |

Low-confidence items get Phase 3 anti-pattern scrutiny but do **not** get embedded as instructions in the optimized prompt body — instead, route them through the optimized prompt as "verify against current docs" guidance, or escalate to a Phase 2.5 user-clarification question.

## Step 1.5D — Integration gate

Before Phase 1.5 hands off to Phase 4 (via Phases 2 / 2.5 / 3), verify the following. Failing any item returns Phase 1.5 to remediation; passing all proceeds.

| Gate | Pass criterion |
|------|----------------|
| **Sourcing** | Every delta item has a source URL and trust tier. |
| **Platform coverage** | At least two platform-unique features surfaced in the delta. |
| **Deprecation flagging** | Every deprecated/superseded pattern is marked for Phase 3 anti-pattern application. |
| **Confidence stamps** | Every item carries High / Medium / Low. |
| **Date stamp** | The delta carries `accessed [session date]` at the top. |
| **No-fabrication check** | Items represent verified findings, not inferred extrapolations. |
| **Feed-readiness** | Phase 4 can pull from the delta without re-reading the underlying sources. |

**Remediation on gate failure.** If a gate fails, re-run the specific gate's source query with a Query 5 form scoped to the gate's deficit (e.g., if "Platform coverage" fails, fire Query 5 against the deployment target until two platform-unique features surface). Cap at 2 remediation iterations. On a third failure, escalate via the Phase 5 delta with an explicit confidence-degradation notice — do not block Phase 4 indefinitely.

## Fallback chain — when calibration cannot be completed

These options are ordered from most disruptive (item 1 — full halt) to least disruptive (item 5 — per-query recovery). When multiple preconditions are met simultaneously, prefer the highest-numbered (least disruptive) option — it preserves the most calibration coverage. Item 1 fires only when no localized recovery is possible.

1. **Network or search-tool unavailable.** Halt Phase 1.5 and surface the failure to the user in the chat run sheet. Do **not** synthesize a delta from memory. Mark the Phase 6A QC summary's "Phase 1.5" row as `FAIL — research unavailable`, and route through to Phase 4 only with the user's explicit acknowledgment that the delivered prompt may reflect stale baselines.
2. **Authoritative sources unavailable but lower-trust sources accessible.** Run the queries against available sources, flag every claim with its true trust tier, downgrade every delta-item confidence by one tier, and surface the degraded-confidence sweep in the chat run sheet.
3. **Source content is sparse — Queries 1–4 yield no new information.** Confirm baseline is current via the "No-change confirmations" bucket. The delta is valid; the absence of change is the result. Do not invent change to populate the other buckets.
4. **One query consistently returns 404 / redirects to a moved URL.** Follow the redirects to the new canonical home (e.g., `docs.anthropic.com` → `platform.claude.com` for some API pages, `docs.claude.com` → `code.claude.com` for Claude Code pages) and re-run. Note the redirect path in the source URL.
5. **Cyclic redirects or completely unresolvable URLs.** Treat that query as failed for this run, fall back to Query 5 with an alternative phrasing, and flag the broken URL in the chat run sheet so the user can update any stored bookmarks.

## Non-interactive execution

In a CLI `-p` (headless) or subagent context where no user is reachable, Phase 1.5 still runs — but its findings are surfaced in the chat run sheet equivalent (whatever the subagent's parent receives as output) rather than confirmed interactively. The Phase 6A QC summary always includes the calibration delta; non-interactive mode does not authorize skipping calibration.

## What this reference is not

This file documents the skill's runtime calibration protocol. It is **not** the build-time research log for the skill itself. When a new build of this skill is produced, the build process may run additional standing queries against the same protocol as a build-time safeguard. Those build-time queries are documented in the build report, not here, because they are not part of the skill's runtime behavior.

## Worked example — Phase 1.5 sweep for a Claude Code CLI / orchestrated-research draft

Draft: orchestrator for a diligence-brief workflow on the Claude Code CLI; the draft references `prompt caching` as a feature.

**Queries fired:**

```
Q1: Claude Opus 4.7 Claude Code CLI best practices 2026
Q2: Anthropic prompt engineering orchestrated research prompt caching
Q3: Claude Opus 4.7 Claude Code CLI configuration
Q4: Anthropic Claude flagship Opus model card 2026
Q5 (conditional — fired because draft names "prompt caching"): Anthropic prompt caching Claude Code CLI 2026
```

**Resulting delta — no-drift case** (the common case; demonstrates that the broadened Query 4 produces a NO-CHANGE confirmation when the flagship is current):

```
PHASE 1.5 CURRENT PRACTICES DELTA — accessed 2026-05-20

NEW / UPDATED PRACTICES
- (none surfaced)

PLATFORM-SPECIFIC CONSIDERATIONS
- CLAUDE_CODE_SUBAGENT_MODEL env var precedence (top precedence over per-invocation and per-subagent frontmatter)
  - Source: https://docs.claude.com/en/docs/claude-code/sub-agents (Tier 1) | Confidence: High | Affects: scope-changing
- Prompt-caching 5-minute TTL for Claude Code CLI sessions (informs subagent-prompt stability)
  - Source: https://docs.claude.com/en/docs/claude-code/prompt-caching (Tier 1) | Confidence: High | Affects: content-changing

DEPRECATED / SUPERSEDED PATTERNS
- (none surfaced)

NO-CHANGE CONFIRMATIONS
- Flagship currency confirmed: claude-opus-4-7 is still the named current flagship; no model-version substitution required (Query 4 broadened, no-drift case)
  - Source: https://docs.anthropic.com/en/docs/about-claude/models (Tier 1) | Confidence: High
- xhigh effort remains the Claude Code CLI default on Opus 4.7
  - Source: https://docs.claude.com/en/docs/claude-code/model-config (Tier 1) | Confidence: High
```

The Phase 6A QC summary's Phase 1.5 row reads PASS: calibration current, model-version substitution did not fire, no stale patterns detected.

## Sources

The query set, source-trust hierarchy, and delta structure are derived from [opus-4-7-config.md](opus-4-7-config.md)'s configuration sweep (accessed 2026-05-20) and from `SKILL.md` Step 1.5B's own specification. The fallback chain reflects observed Anthropic doc-host migrations (`docs.anthropic.com` → `platform.claude.com`, `docs.claude.com` → `code.claude.com`) as of 2026-05-20.

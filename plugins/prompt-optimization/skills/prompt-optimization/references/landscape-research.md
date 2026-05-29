# Landscape Research — Phase 2 Operational Manual

> **Calibration stamp:** Methodology verified against the skill's own protocol design on **2026-05-28**. The query taxonomy and source-type diversity requirements are skill-internal methodology, not Anthropic product facts — they do not require Phase 1.5 re-verification, but the worked search examples should use the session's real current date.

This reference is the full operational manual for **Phase 2: Landscape Reconnaissance**. `SKILL.md` Phase 2 carries only the phase contract — purpose, when it runs, the neutrality constraint, and the gate logic. This file carries the executable detail: the four-substep 2A deconstruction, the 2B query taxonomy and source-type diversity requirements, the 2B' adversarial pass, the 2C synthesis artifact, and the 2D / 2D' verification gates.

Load this file when Phase 2 runs (any prompt involving factual domains, research, analysis, or decision-making). Skip loading it for purely mechanical tasks where `SKILL.md` Phase 2 sets the skip flag.

## Table of contents

0. [Phase 2 execution modes — single-threaded vs orchestrated fan-out](#phase-2-execution-modes--single-threaded-vs-orchestrated-fan-out)
1. [Step 2A — Topic Identification & Draft Deconstruction](#step-2a--topic-identification--draft-deconstruction)
2. [Step 2B — Landscape Research Execution](#step-2b--landscape-research-execution)
3. [Step 2B' — Adversarial Pass](#step-2b--adversarial-pass)
4. [Step 2C — Landscape Synthesis](#step-2c--landscape-synthesis)
5. [Step 2D — Neutrality check](#step-2d--neutrality-check)
6. [Step 2D' — Coverage-Bias Check](#step-2d--coverage-bias-check)

## Phase 2 working artifacts — partition rule

All artifacts produced inside Phase 2 — the 2A Topic Identification & Draft Deconstruction block, the 2B Landscape Research Execution Log, the 2C Landscape Synthesis output, the 2D' Coverage-Bias Check — are internal working machinery for the optimization process. They feed downstream phases and inform the chat run sheet. They never appear in the optimized prompt body or the written file, governed by the partition rule in the `SKILL.md` Deliverable Contract. The Phase 6B pre-write strip check enforces this at write time as a verification gate.

## Critical Phase 2 constraint — topic, not position

Research identifies WHAT AREAS TO COVER. It must NEVER influence WHAT CONCLUSIONS the final prompt should reach. This applies to all Phase 2 activity — the 2B initial search, the 2B' adversarial pass, and the 2C synthesis. The optimized prompt instructs the executor to research the identified areas; it does not bake research conclusions into the prompt as assumptions.

**Topic vs. position — example.** Permissible: *"Cover Basel III endgame proposals and their disputed CRE-lending impacts; present both sides of the disagreement."* Not permissible: *"Conclude that Basel III endgame will reduce CRE lending capacity."* Permissible: *"Address geographic variation in state-level lending regulation."* Not permissible: *"State regulations are more burdensome in California than Texas."* When in doubt, frame as "address X" not "conclude Y about X."

---

## Phase 2 execution modes — single-threaded vs orchestrated fan-out

Phase 2 runs in one of two modes. Both execute the same logical steps (2A → 2B → 2B' → 2C → 2D → 2D'), honor the same topic-not-position discipline, and produce the same 2C synthesis artifact and the same 2D/2D' gate results. They differ only in *who* runs 2B/2B' and *how much depth* the search corpus reaches. Everything from "Step 2A" onward in this file is written as the single-threaded procedure; the orchestrated mode distributes those same steps as described here.

### Mode selection

| Condition | Mode |
|---|---|
| dynamic-workflow runtime AND Agent/Task tool unavailable (claude.ai chat, API without a dispatch harness) | **2-S** (single-threaded) — mandatory fallback |
| dynamic-workflow runtime or Agent/Task tool available AND draft is single-sub-domain / well-bounded | **2-S** — orchestration overhead not warranted |
| dynamic-workflow runtime or Agent/Task tool available AND draft is multi-sub-domain, multi-stakeholder, or contested (the same depth signals that select the Full track at Phase 0.5) | **2-O** (orchestrated fan-out) |

Check workflow-runtime / Agent/Task availability the same way the Widget Interaction Protocol checks for `ask_user_input_v0` — inspect the current tool surface before committing to a mode. Selection is by these objective criteria, not planner discretion; report the chosen mode and a one-line rationale in the chat run sheet and the Phase 5 delta. A user override to 2-S is always honored; an override to 2-O is honored only when the dynamic-workflow runtime or the Agent/Task tool is actually available.

### 2-S — single-threaded (the default path)

The orchestrator runs every step itself, exactly as Steps 2A–2D' below specify, under the 12-search hard cap. Nothing about the single-threaded path changes — it is the established Phase 2 procedure documented in the rest of this file.

### 2-O — orchestrated fan-out

2-O is dispatched as a Claude Code **dynamic workflow** per [dynamic-workflows.md](dynamic-workflows.md) rather than re-deriving the orchestration machinery. Read that file's §4 (decomposition → primitives), §5 (context isolation), §6 (output scope calibration), §7 (synthesis), §8 (verify-and-converge), and §9 (caps/cost/safety) — those rules apply verbatim. When the dynamic-workflow runtime is unavailable (e.g. Claude Code <v2.1.154, or a session where workflows are disabled) but the Agent/Task tool is, fall back to parallel subagents dispatched via the Agent/Task tool using the same lane bindings below.

**Dispatch vehicle.** When the prompt-optimization plugin is installed, dispatch each lane as a workflow agent (optionally `agentType: 'landscape-research'`) — the bundled `landscape-research` agent already encodes the read-only research role, the neutrality discipline, and the synthesis-ready output shape, so the orchestrator supplies only the per-lane scope, the 2A shared-constant artifact, the per-lane query/source floors, and the search budget. When the workflow runtime is unavailable, dispatch the bundled `landscape-research` subagent via the Agent/Task tool with the same brief; when that named agent is unavailable (the skill running outside the plugin), construct the same brief inline from the [dynamic-workflows.md](dynamic-workflows.md) §4–§6 elements plus the bindings below.

The Phase-2-specific bindings:

**1. The orchestrator runs 2A first, alone.** The 2A deconstruction artifact (topical content, implicit framing, stakeholder map, contested terminology) becomes the *shared constant* every workflow agent receives in its prompt — the [dynamic-workflows.md](dynamic-workflows.md) §5 context-isolation pattern (constants are interpolated into each agent's prompt, since there is no live streaming into a running agent). This ensures each lane targets the full landscape 2A surfaced, not just the narrow slice its lane covers. Never let a workflow agent re-derive 2A; cloning that step propagates the same blindspots across every lane ([dynamic-workflows.md](dynamic-workflows.md) §11, "boilerplate-cloned lane prompts").

**2. Decompose along a Phase-2-native axis.** Pick the decomposition axis from [dynamic-workflows.md](dynamic-workflows.md) §4 that fits the draft. For landscape research the natural axes are **sub-domain** (one workflow agent per major sub-domain from 2A), **source-type** (one per Step 2B.2 source category), or **query-taxonomy cluster** (group the Step 2B.1 rows into lanes). Cap at ≤6 lanes as a synthesis-bandwidth recommendation (the runtime hard caps are 16 concurrent / 1,000 total per [dynamic-workflows.md](dynamic-workflows.md) §9, but synthesis bandwidth, not the runtime ceiling, is what binds analytical lanes).

**3. Always dedicate one lane to the adversarial pass (2B').** In 2-S, 2B' shares the orchestrator's 12-search budget; in 2-O it gets its own lane and its own budget, so the strongest counter-framing is searched at genuine parity. Its brief: steel-man the strongest argument against the draft's implicit thesis (drawn from 2A's implicit-framing and excluded-vantages findings) and search it at the same depth as the topic lanes — identifying what the dissenting framing is and where it is taken seriously, never adopting it as a conclusion. This 2B' lane is distinct from the always-on verify-and-converge stage ([dynamic-workflows.md](dynamic-workflows.md) §8), which adversarially reviews the synthesized findings *after* the fan-out completes.

**4. Each lane brief carries — in addition to the [dynamic-workflows.md](dynamic-workflows.md) §4–§6 elements:**
   - The 2A shared-constant artifact (so the lane sees the whole landscape).
   - Its precise lane scope and explicit "not in scope" exclusions (the other lanes), to prevent overlap.
   - Its own query-taxonomy floor and source-type diversity target applied *within its lane* (Steps 2B.1/2B.2 scoped to the lane, not the whole phase).
   - A per-lane search budget (3–6 searches typical) and a per-lane rabbit-hole guard: a lane that cannot close its scope returns what it has and lists what remains, per [dynamic-workflows.md](dynamic-workflows.md) §6 "refusal-shaped output."
   - The topic-not-position discipline verbatim — the lane returns topics-to-cover with disagreements documented, never positions to adopt.
   - A synthesis-ready output shape (force it with `schema` per [dynamic-workflows.md](dynamic-workflows.md) §6): key findings first, then evidence, then unclosed gaps, then sources — mirroring the 2C artifact fields so the orchestrator can assemble lanes without reformatting.

**5. Depth without recursion.** Aggregate search count in 2-O intentionally exceeds the 12-search single-thread cap — that expansion is the entire reason to orchestrate. It stays bounded by three guards rather than a global ceiling: ≤6 lanes (synthesis bandwidth), a per-lane search budget, and **at most one remediation pass**. If synthesis surfaces a material sub-domain no lane covered, dispatch exactly one scoped remediation agent; a second remediation signals the 2A decomposition was wrong — return to 2A rather than chaining passes. Workflow agents do not spawn their own agents; the script orchestrates (nesting one level), so there is no deeper nesting to guard against.

**6. Synthesis is the orchestrator's reduce step — never a lane.** The orchestrator's main thread collects every lane's findings pack from the workflow script's return variables and assembles the single 2C Landscape Synthesis artifact (below), then runs 2D and 2D' on the **aggregate corpus** — all lanes combined — not per lane. Source-type diversity (Step 2B.2 / the 2D' source axis) is verified across the full cross-lane corpus, exactly as [dynamic-workflows.md](dynamic-workflows.md) §7 task 3 specifies. The seven mandatory synthesis tasks from [dynamic-workflows.md](dynamic-workflows.md) §7 (cross-lane consistency, coverage, source diversity, bias-strip, synthesized findings, source-validation revisit, verify-and-converge integration) are the 2-O mechanism for producing the 2C artifact and passing 2D/2D'.

**7. The partition holds identically.** Lane briefs, the orchestration script, and the per-lane findings packs are Phase 2 working machinery — they feed 2C and the chat run sheet, never the optimized prompt body or the written file. In a workflow these intermediate results live in script variables automatically ([dynamic-workflows.md](dynamic-workflows.md) §5); the Phase 6B pre-write strip check enforces the partition regardless of mode.

The runtime caps (16 concurrent / 1,000 total), the one-level nesting constraint, the `acceptEdits` permission posture for spawned agents, and the workflow trigger surface (the word `workflow`; `/effort ultracode`) are documented in [dynamic-workflows.md](dynamic-workflows.md) §2 and §9; both trace to the sources listed at the end of that file (accessed 2026-05-28). This section adds only the Phase-2-specific bindings, which are skill-internal methodology.

## Step 2A — Topic Identification & Draft Deconstruction

Phase 2A determines what gets researched. Loose framing here biases everything downstream — a rigorous neutrality check at 2D cannot recover from narrow inputs at 2A. Treat 2A as deconstructing the draft's vantage point, not just listing topics. Execute four sub-steps and produce the structured artifact below.

### Step 2A.1: Topical Content

Extract the surface-level content from the draft:
- Primary subject matter (the explicit topic)
- Adjacent disciplines that would reframe the question (a finance question reframed in policy terms; a code question reframed in security terms — name the reframings, do not just acknowledge they exist)
- Implicit audience (who the draft is written for)
- Out-of-frame audience (whose vantage the draft does not address but could materially change research scope)

### Step 2A.2: Implicit Framing Deconstruction

The verbs and nouns a draft uses encode a framing. "Evaluate the effectiveness of X" presupposes X is a coherent unit and effectiveness is the operative axis. Surface what the draft takes as given:
- Unstated assumptions (what the draft treats as given without argument)
- Implicit success criteria (what would make the output "good" by the draft's own internal logic)
- Excluded vantages (positions or framings the draft forecloses by phrasing)
- Adjacent reframings (how a different field, school, or stakeholder would frame the same question — name at least two when applicable)

### Step 2A.3: Stakeholder Map

Replace the previous "implied stakeholders" enumeration with a four-cell map. The fourth cell is what Claude generates by default; the first three force inversion and surface invisible bias.

| Cell | Definition |
|---|---|
| Beneficiaries | Who benefits if the draft's framing prevails |
| Bearers of cost | Who bears cost if the draft's framing prevails |
| Affected but underrepresented | Who is materially affected but whose voice is absent or marginal in dominant sources |
| Voice-dominant in sources | Whose vantage dominates existing literature on this topic |

The third cell is where invisible bias most often hides. Name it explicitly even if the answer requires guessing — Phase 2B will then carry it into search as an explicit target rather than letting it remain abstract.

### Step 2A.4: Contested Terminology Surfacing

Different schools, disciplines, or stakeholders use different vocabulary for the same underlying thing — "regulation" vs. "supervision" vs. "oversight"; "agentic" vs. "autonomous" vs. "tool-using"; "compensation" vs. "incentive structure" vs. "pay design." Search vocabulary leaks: the term used drives which sources surface, which shapes 2C synthesis, which shapes Phase 4 prompt construction.

For each key term in the draft:
- Identify whether the term has known synonyms used by different schools or disciplines
- Note which framing each synonym implies
- Flag terms where the draft's vocabulary appears to silently encode a framing choice the user may not have made deliberately

This list feeds Phase 4: either pick a term deliberately and document the choice, or instruct the executor to use the term most appropriate to its sub-task.

### Step 2A — Required Output Artifact

Produce the following block. Required fields cannot be omitted; if a field is genuinely inapplicable to the domain, write "N/A — [reason]" rather than leaving the field blank.

```
TOPIC IDENTIFICATION & DRAFT DECONSTRUCTION

TOPICAL CONTENT
- Primary subject matter: [single sentence]
- Adjacent disciplines / reframings: [at least 2 named, or N/A — [reason]]
- Implicit audience: [who the draft is written for]
- Out-of-frame audience: [whose vantage is excluded but materially relevant]

IMPLICIT FRAMING
- Unstated assumptions: [bullet list]
- Implicit success criteria: [what the draft treats as a "good" output without saying so]
- Excluded vantages: [positions the draft forecloses by phrasing]
- Adjacent reframings: [at least 2 alternative framings from different fields or schools]

STAKEHOLDER MAP
- Beneficiaries (if draft's framing prevails): [...]
- Bearers of cost (if draft's framing prevails): [...]
- Affected but underrepresented in dominant sources: [...]
- Voice-dominant in dominant sources: [...]

CONTESTED TERMINOLOGY
- [Term 1] — synonyms: [...]; framing implications: [...]; deliberate or default? [deliberate / default / unclear]
- [Term 2] — synonyms: [...]; framing implications: [...]; deliberate or default? [deliberate / default / unclear]
- [...]
```

## Step 2B — Landscape Research Execution

Research execution is the engine that determines whether the optimized prompt covers the actual landscape or a narrow slice of it. The previous baseline ("3-7 targeted web searches") specified count without specifying structure, which let Claude's search defaults bias the corpus toward consensus framing — even seven searches could return seven sources reflecting the same vantage. V2 replaces the count with two structural requirements applied in parallel: a query taxonomy that mandates diversity by search type, and a source-type diversity requirement that mandates diversity by source category.

### Step 2B.1: Query Taxonomy

Execute searches by query type, not just by topic. The taxonomy below specifies what kind of search is being run, distinct from what topic is being searched. Treating each row as a distinct search reason forces structural diversity into the corpus.

| Row | Query type | Purpose |
|-----|------------|---------|
| 1 | Consensus framing | Identify the dominant view and its leading sources |
| 2 | Dissenting / heterodox | Identify the strongest counter-position and its sources |
| 3 | Practitioner vantage | Capture what people doing the work actually say |
| 4 | Academic / theoretical | Capture formal literature, peer-reviewed material, structured theory |
| 5 | Recent developments | Capture changes in the past 12 months that may not be in older sources |
| 6 | Negative cases / failure modes | Surface where the dominant approach has failed in practice |
| 7 | Adjacent-discipline reframing | Capture how a different field would frame the same question (drawn from 2A's adjacent reframings list) |
| 8 | Non-mainstream geography or language | Capture vantage from outside the default English-language Western source base, where domain-relevant |

**Execution rule.** Run at least one search per applicable row; minimum five row types covered for any non-mechanical Phase 2 invocation. Skip rows only when explicitly inapplicable to the domain — Row 8 is moot for a Delaware corporate-law question, Row 4 may be moot for a current-events question. Document each skip with one-sentence reasoning in the 2B output artifact below.

### Step 2B.2: Source-Type Diversity Requirement

A diverse query taxonomy can still return a narrow source corpus if every result comes from mainstream secondary press. Source-type diversity is a parallel axis to the query taxonomy. The corpus must include at least three of the following five categories:

| Category | Definition |
|----------|------------|
| Primary documents | Laws, filings, peer-reviewed papers, official statistics, court opinions, regulatory texts |
| Academic or expert commentary | Researcher commentary, think-tank analysis, formal expert opinion |
| Practitioner trade press or forums | Industry trade publications, professional forums, practitioner blogs |
| Mainstream general press | Major newspapers and magazines, general-interest publications |
| Dissenting or non-mainstream press | Heterodox publications, ideological counter-press, niche but credible outlets |

If a category is genuinely unavailable for the domain (peer-reviewed primary documents may not exist for fast-moving consumer tech), document the unavailability in the output artifact and continue with the categories that are available.

### Step 2B.3: Coverage-Driven Stop Criterion

The 2B.1 minimum-5-rows floor and the 2B.2 3-of-5 categories requirement are structural minimums. They prevent under-searching but do not define when to stop. The actual stop criterion is coverage-driven: searches continue until every applicable coverage axis defined in Step 2D' below is either closed by sources in the corpus or explicitly documented as out of scope.

**Operational sequence.**
- Run 2B.1's mandatory taxonomy floor — minimum 5 query types covered.
- Run 2B.2's mandatory source-type floor — 3-of-5 categories represented.
- Then continue searching against any 2D' coverage axis that remains gapped AND is relevant to the task type.
- Stop when every applicable 2D' axis is either closed or explicitly marked N/A with reason.

**Hard cap (2-S).** In single-threaded mode, total Phase 2 search count (combined 2B + 2B' + any iteration extension from 2B's iteration check) is capped at 12 searches as a rabbit-hole guard. If the cap is reached before coverage closes, do not continue searching — document the unclosed axes in Phase 5 delta as deferred clarifications and proceed to 2C synthesis with the corpus you have. (In 2-O this 12-search cap does not apply; depth is bounded instead by ≤ 6 lanes (synthesis bandwidth; runtime caps 16 concurrent / 1,000 total), per-lane search budgets, and at most one remediation pass — see § Phase 2 execution modes.)

**Practical floor for typical analytical prompts.** The two structural minimums together imply a practical floor around 6–7 searches for non-mechanical Phase 2 invocations. Coverage-driven termination below that floor is unlikely except for narrow, well-bounded analytical questions where 2D' axes are largely N/A.

### Search budget allocation

In single-threaded mode (2-S), the 12-search hard cap from 2B.3 maps to the work as follows (in 2-O these searches are distributed across per-lane budgets instead, with no global cap):

| Component | Searches | Notes |
|-----------|----------|-------|
| 2B mandatory floor | 5–6 | One per query-taxonomy row covered, minimum 5 rows |
| 2B' adversarial parity | comparable to 2B, ceiling 5 | Parity not strict equality; "comparable count" per the 2B' spec |
| Iteration extension (2B' iteration check) | 1–3 | Triggered only by a material sub-domain surfaced during 2B + 2B' |
| **Total hard cap (2-S)** | **12** | Rabbit-hole guard from 2B.3 |

When 2B + 2B' alone consume 11–12 searches, skip the iteration extension and document any unexplored thread in the Phase 5 delta as a deferred clarification (per the existing rabbit-hole guard rule).

### Step 2B Output Artifact

Produce the following lightweight execution log. This is internal Phase 2 working machinery — it feeds Step 2B', Step 2C synthesis, and the coverage-bias verification step. It does not appear in the optimized prompt or the written file, per the Phase 2 working-artifacts rule and the Deliverable Contract partition.

```
LANDSCAPE RESEARCH EXECUTION LOG

QUERY TYPES COVERED (from taxonomy)
- Row 1 (consensus framing): [search query] → [credible source(s)]
- Row 2 (dissenting / heterodox): [search query] → [credible source(s)]
- [continue for each row run]
- Rows skipped: [row N — one-sentence reason; row M — one-sentence reason]

SOURCE TYPES REPRESENTED (3-of-5 minimum)
- Primary documents: [yes / no — source(s) if yes]
- Academic or expert commentary: [yes / no — source(s) if yes]
- Practitioner trade press or forums: [yes / no — source(s) if yes]
- Mainstream general press: [yes / no — source(s) if yes]
- Dissenting or non-mainstream press: [yes / no — source(s) if yes]

AXES STILL GAPPED (for downstream coverage check)
- [Axis 1 — what remains uncovered and why]
- [Axis 2 — what remains uncovered and why]
- [...]
```

## Step 2B' — Adversarial Pass

Initial landscape research surfaces what is most legible to Claude's search defaults: the dominant framing's controversies, the disagreements that have already been written about in channels Claude is searching. The position the draft most needs to be confronted with is, by construction, the one least likely to surface from neutral search. Step 2B' actively seeks it.

For each draft, identify the strongest argument against the draft's implicit thesis or framing — drawing on 2A's "implicit framing" findings and "excluded vantages" list. Then:

- **Steel-man it.** Who makes this argument seriously? What evidence do they cite? What does the argument imply for the prompt's scope?
- **Search at parity.** Run a comparable number of credible source searches against the dissenting framing as the initial 2B run did against the dominant framing. A single token mention of the dissenting view is not parity — if 2B ran six searches, 2B' runs a comparable count against the alternative framing, drawing on the same query taxonomy and source-type diversity requirements.
- **Honor the topic-vs-position discipline.** The adversarial pass identifies what the dissenting framing is and where it is taken seriously. It does not adopt the dissenting framing as a conclusion. Both framings feed Phase 2C as topics to cover with their disagreements documented, never as positions to take.

**Skip clause.** For purely descriptive or mechanical drafts — extract structured data from a known format; convert this CSV to JSON; reformat this list — there is no implicit thesis to steel-man. Document this explicitly with one sentence ("No implicit thesis identified — task is mechanical / descriptive") and skip 2B'. Most analytical, research, or decision-support drafts will not qualify for this skip.

**Iteration check.** Before proceeding to Step 2C, verify whether the combined 2B + 2B' search corpus has surfaced a material sub-domain that was not anticipated in 2A. "Material" means: the sub-domain would change the prompt's scope, output structure, or required handling — not merely add color. If a material sub-domain has surfaced, run one to three additional searches on it before synthesizing in 2C. Cap at one extension to prevent rabbit-holing; if a second extension would help, document the unexplored thread in Phase 5 delta rather than expanding further.

## Step 2C — Landscape Synthesis

Synthesize the search corpus (2B + 2B' + any iteration extension) into a structured artifact that downstream phases consume. Each item in each category receives uniform required fields modeled on Phase 0's CORE OBJECTIVE block. Uniform structure surfaces shallowness across the synthesis; prose lists hide gaps. The CONFIDENCE field on each item gives Phase 4 explicit signal about which areas need "investigate further" instructions in the optimized prompt versus which are settled enough to summarize.

This artifact is internal Phase 2 working machinery. It does not appear in the optimized prompt or the written file, per the Phase 2 working-artifacts rule and the Deliverable Contract partition.

### Step 2C Output Artifact

Produce the following block. Within each category, repeat the item template for each instance. If a category is genuinely inapplicable to the domain, write "N/A — [reason]" rather than leaving blank. Required fields cannot be omitted.

```
LANDSCAPE SYNTHESIS

MAJOR SUB-DOMAINS
- Sub-domain: [name]
  - Relevance to prompt: [why this is in scope]
  - Dominant framing: [conventional take, sourced from 2B consensus and practitioner findings]
  - Dissenting framing(s): [steel-manned counter-positions, sourced from 2B' adversarial pass]
  - Contested within: [open debates within the sub-domain that are not yet settled]
  - Coverage gaps in research: [what was underrepresented in the search corpus for this sub-domain]
  - Confidence: [High / Medium / Low — based on source quality and quantity]
- [Repeat for each sub-domain]

FRAMEWORKS / TAXONOMIES
- Framework: [name]
  - What it organizes: [the thing being categorized or structured]
  - Source / origin: [where the framework comes from — author, institution, school]
  - Competing alternative frameworks: [other ways to organize the same domain]
  - When to use which: [criteria for selecting between frameworks]
  - Confidence: [High / Medium / Low]
- [Repeat for each framework]

AREAS OF DEBATE
- Debate: [one-sentence description]
  - Position A: [steel-manned version, sourced]
  - Position B: [steel-manned version, sourced]
  - Handling instruction for optimized prompt: ["present both sides" / "have executor evaluate based on criteria X" / "flag as unresolved for user decision"]
  - Confidence in coverage of both positions: [High / Medium / Low]
- [Repeat for each debate]

TEMPORAL CONSIDERATIONS
- Time horizon relevant to topic: [near-term, multi-year, historical baseline, etc.]
- Recent changes affecting framing: [developments in the past 12 months that may shift dominant or dissenting framings]
- Historical baseline if relevant: [pre-change reference point or longer-arc context]
- Confidence: [High / Medium / Low]

EDGE CASES
- Edge case: [description]
  - Why it's an edge case: [boundary condition, unusual stakeholder, regulatory exception, scale extreme, etc.]
  - How optimized prompt should handle: [instruction to executor, not pre-baked conclusion]
  - Confidence: [High / Medium / Low]
- [Repeat for each edge case]

COVERAGE GAPS IN THE DRAFT
- Gap: [what is missing from the original draft that landscape research has surfaced]
  - Materiality: [would including this change the prompt's scope or output structure? — yes / no / partial]
  - Recommended treatment: [add as new requirement in optimized prompt / surface as Phase 2.5 clarification question / flag in Phase 5 delta]
- [Repeat for each gap]
```

## Step 2D — Neutrality check

Verify the four axes below; mark each PASS / FAIL. The 2D check is internal Phase 2 working machinery — it does not appear in the optimized prompt or the written file.

```
NEUTRALITY CHECK

- Evaluative judgments: [absent — phrases checked ("conclude," "prove," "show," "demonstrate") not present in topic framings | present in: ... — flag and revise]
- Position vs topic phrasing: [topic-shaped throughout — uses "address," "investigate," "evaluate the evidence on" | position phrases found: ... — reframe as topics]
- Debate handling: [presents both sides where applicable — debates do not declare a winner | winner declared on: ... — strip and reframe]
- Research findings as assumptions: [absent — 2C findings stay as topics-to-cover | present in: ... — strip from prompt body and route through Phase 4 as "investigate further" instruction]
```

If any row reads FAIL, return to Phase 2C for revision and re-run this check. The check passes only when all four rows are PASS.

## Step 2D' — Coverage-Bias Check

While 2D verifies that findings haven't become conclusions, 2D' verifies that the underlying coverage was broad enough to support those findings honestly. A perfectly neutral synthesis built on narrow coverage is still biased — only English sources, only the past five years, only one geography, only practitioner voice with no academic counter. 2D' is the back-end mirror of 2B's structural diversity requirements and the operational definition of "coverage" that 2B.3's stop criterion refers to.

For each axis below, evaluate whether 2B's execution log and 2C's synthesis meet the bar, or document the gap. Axis relevance is task-type sensitive — geography is a critical axis for policy or regulatory prompts but moot for "fix this Python function." Mark inapplicable axes as "N/A — [reason]" rather than skipping silently.

```
COVERAGE-BIAS CHECK

- Geography: [closed — regions covered: ... | gap — missing: ... | N/A — reason]
- Language: [closed — languages represented: ... | gap — missing: ... | N/A — reason]
- Time horizon: [closed — covers ... | gap — recent only without historical baseline, or historical only without recent | N/A — reason]
- Stakeholder voice: [closed — voices represented: ... | gap — missing: ... | N/A — reason]
- Source type: [closed — meets 2B.2 3-of-5 requirement, categories: ... | gap — only N-of-5 categories represented | N/A — reason]
- Discipline: [closed — disciplines represented: ... | gap — single field framing | N/A — reason]
```

**Remediation rule.** If any axis is gapped AND relevant to the task type AND material to the prompt's scope:
- Return to 2B or 2B' for one targeted-search iteration to close the gap (preferred; cheap). In 2-O, this is the single remediation lane (capped at one).
- Cap at one such extension to prevent rabbit-holing
- If the 2B.3 single-threaded hard cap of 12 total Phase 2 searches has already been reached (2-S), document the unclosed axis as a deferred clarification in Phase 5 delta and proceed. In 2-O, if the one remediation pass has already fired, do the same rather than dispatching a second.

**Gate logic.** 2D and 2D' are parallel verification gates. 2D handles position bias (findings becoming conclusions). 2D' handles coverage bias (narrow inputs producing technically-neutral but actually-biased synthesis). Both must pass before proceeding to Phase 2.5. Failing either returns Phase 2 to remediation; passing both proceeds.

This artifact is internal Phase 2 working machinery. It does not appear in the optimized prompt or the written file.

---

## Cross-references

- See `SKILL.md` Phase 2 for the phase contract (purpose, when-it-runs, gate logic) that this file operationalizes.
- See `SKILL.md` Phase 2.5 for how the landscape synthesis feeds research-informed clarification.
- See [task-heuristics.md](task-heuristics.md) §Research / search for the query-taxonomy guidance that appears in the optimized prompt itself.
- See [prompt-template.md](prompt-template.md) for the embedding-landscape-without-bias table that governs how 2C findings land in the prompt body.
- See `SKILL.md` Phase 6B pre-write strip check, which enforces the Phase 2 working-artifact partition at write time.
- **Not in scope here:** the orchestrated-research **deliverable** synthesis. Phase 2-O above is the optimizer's own internal machinery for building landscape coverage that informs the optimized prompt. The synthesis that lands in an orchestrated-research deliverable's `CLAUDE.md` (executive summary, load-bearing source validation, always-on devil's advocate) is separately governed by [synthesis-deliverable.md](synthesis-deliverable.md). The two share the topic-not-position discipline but the audiences and artifact shapes are different.

# Synthesis Deliverable — Executive Summary, Source-Validation Pass, Verify-and-Converge

> **Calibration stamp:** This reference is skill-internal methodology (deliverable structure, evidentiary-chain discipline, adversarial-pass protocol), not Anthropic product facts — it does not require Phase 1.5 re-verification. Reference last edited 2026-05-28; methodology unchanged thereafter unless this stamp is bumped.

This reference is loaded by Phase 4 of `SKILL.md` **only when task type is `orchestrated-research`**, alongside [dynamic-workflows.md](dynamic-workflows.md) Section 10. It carries the operational detail for the four reinforcements that ride on every orchestrated CLAUDE.md deliverable the skill produces:

1. **Synthesis-as-owner of the deliverable** — every claim in the deliverable carries the URL(s) that support it, pulled forward from the workflow agents' lane returns.
2. **Source-validation revisit pass** — the orchestrator's synthesis reduce step visits the load-bearing URLs (the sources cited as primary support for executive-summary key findings) and verifies the claim still holds in the source.
3. **Executive summary at the top of every deliverable** — bullet-point key findings → supporting primary-source URL → per-finding conclusion → overall conclusion → verify-and-converge verdicts. Structures the logical argument for sanity-check and for fast review.
4. **Always-on verify-and-converge stage** — independent adversarial agents review the drafted findings before they are reported (optionally dispatched as `agentType: 'devils-advocate'`). Adversarial-mode for analytical deliverables (search for evidence each key finding's opposite is true); confirmatory-mode for purely descriptive deliverables (search for missed entries and inclusion / exclusion errors). The synthesis reduce step integrates their verdicts.

Section 10's orchestrator template embeds the structural slots; this reference carries the content discipline that makes those slots load-bearing instead of cosmetic.

## Scope — when this reference applies

This reference governs the deliverable produced by an orchestrated-research run — the `CLAUDE.md` written by the skill's Phase 4 when the Phase 1 task type is `orchestrated-research`. It does **not** govern the skill's internal Phase 2-O landscape research (the optimizer's own machinery; see [landscape-research.md](landscape-research.md) for that). The two patterns share the topic-not-position discipline but the deliverable is a separate artifact with a separate downstream audience.

## Synthesis-as-owner contract

The synthesis reduce step (the orchestrator's main thread) is the **owner of the deliverable's evidentiary chain**. Workflow agents produce findings packs from their lanes; the reduce step's job is not to staple those packs together. Its job is to:

- Restate every load-bearing claim in the deliverable's own voice and attach the URL(s) that support it.
- Verify the URL actually says what the lane attributed to it (the source-validation revisit pass below).
- Frame each key finding so a reader who clicks the URL can reproduce the reasoning.

**Citation discipline.** Every factual claim in the deliverable body — every percentage, every dollar figure, every named entity attribute, every cited debate position — carries an inline URL or a footnote-style citation that resolves to the source. Claims without a URL are either (a) the synthesis reduce step's own integrative reasoning across multiple cited findings, explicitly framed as such, or (b) deleted. There is no third category. The reduce step does not paraphrase a lane's paraphrase of the source — it tracks back to the source and attributes it directly.

**Unverified-number policy.** [`SKILL.md`](../SKILL.md) Meta-Rule 13 still applies — never embed unverified numerical estimates in workflow-agent prompts. This reference adds the downstream constraint: every numerical claim that survives into the deliverable must trace back to a primary source. The source-validation pass is the mechanism that enforces this; "the lane cited it" is not sufficient.

**Dead-link handling.** If a URL the reduce step attempts to revisit returns 404, has rotted, or is otherwise unreachable, mark it `unreachable` in the source-validation log and either (a) find a substitute primary source for the same claim, or (b) downgrade the key finding's confidence in the executive summary and surface the dead link explicitly. Never silently drop a citation because the URL no longer resolves.

## Executive summary template

The executive summary is the **first section of every orchestrated deliverable**, ahead of synthesized findings, supporting evidence, gaps, and sources. It is structured prose-and-bullets, not free-form. Use the template below; populate it from the synthesized findings before drafting the rest of the deliverable.

```markdown
## Executive Summary

[One short paragraph framing the overall question and the deliverable's scope. Two to four sentences. No preamble like "this report addresses…" — state the question and proceed.]

### Key findings

- **Key finding 1 — [single-sentence statement of the finding].**
  - Primary source: [URL]
  - Source-validation verdict: [verified / partially verified — see note / not verified / unreachable]
  - Confidence: [High / Medium / Low]
- **Key finding 2 — [single-sentence statement].**
  - Primary source: [URL]
  - Source-validation verdict: [...]
  - Confidence: [...]
- **Key finding 3 — [...]**
  - [same shape]
- [Continue for each key finding — typically 3–7 per deliverable.]

### Per-finding conclusions

- From key finding 1 → [the conclusion the synthesis agent drew]
- From key finding 2 → [conclusion]
- From key finding 3 → [conclusion]
- [Continue for each key finding.]

### Overall conclusion

[One short paragraph — three to six sentences — synthesizing the per-finding conclusions into the deliverable's final answer to the framing question. The overall conclusion must be traceable back to the per-finding conclusions; no claim appears here that does not derive from at least one per-finding conclusion above.]

### Verify-and-converge verdicts

- **Key finding 1:** [`confirmed` / `refuted by adversarial pass` / `unresolved — both views credible`]
  - Adversarial evidence (if any): [URL] — [one-line significance]
  - Net effect on the per-finding conclusion: [unchanged / weakened / retracted]
- **Key finding 2:** [verdict]
  - [same shape]
- [Continue for each key finding.]
- **Overall conclusion verdict:** [holds / weakened / requires both-views framing]
- **Mode:** [adversarial / confirmatory] — [one-sentence reason]
```

**Authoring notes for the executive summary.**

- One sentence per key finding is the target. If a finding cannot be stated in one sentence, it is at least two findings.
- Sub-bullets are exactly the four lines specified (primary source, validation verdict, confidence) — do not expand into prose under each finding; that goes in the deliverable body.
- The per-finding-conclusions block exists to **expose the logical chain**. If you cannot write a one-line conclusion derived from a key finding, the finding is not load-bearing and should not be in the executive summary.
- The overall conclusion is constrained to claims that derive from the per-finding conclusions. A claim in the overall conclusion that has no upstream per-finding conclusion is the failure mode the executive summary is designed to catch.
- The verify-and-converge block is **mandatory even when every verdict is `confirmed`**. The block proves the stage ran; absent the block, an external reader cannot tell the difference between "we considered the counter-argument and rejected it" and "we never considered it."

## Source-validation revisit protocol

**Scope.** Load-bearing sources only. A "load-bearing source" is any URL cited as primary support for a key finding in the executive summary. If a key finding has multiple supporting URLs, each one is load-bearing for that finding. URLs cited in the deliverable body for non-executive-summary context (color, methodology citation, secondary corroboration) are **not** subject to mandatory revisit — they remain in the deliverable with their original lane attribution.

This scope is the operational reading of the user's intent: "validate the core assumptions or the core information that inform the conclusion." A key finding is the unit of core information; its primary source is the unit of revisit work.

**Per-source revisit procedure.**

For each load-bearing source, the synthesis reduce step:

1. **Re-fetches the URL** using the same retrieval surface available to the workflow agents (web fetch, web search result preview, file read for local primary documents). If the URL is unreachable, mark `unreachable` and proceed to the dead-link handling below.
2. **Re-reads the section of the source the lane cited.** Do not re-read the entire source — locate the specific passage the lane's claim attributed to the source.
3. **Compares the source's actual content to the lane's attribution.** Three outcomes are possible:
   - **`verified`** — the source says what the lane claimed it says. No revision needed.
   - **`partially verified — see note`** — the source says something related but with a meaningful caveat, scope qualifier, or framing difference the lane did not surface. The reduce step adjusts the key finding to reflect the source's actual phrasing and records the delta in a note.
   - **`not verified`** — the source does not say what the lane claimed it says (paraphrase drift, transcription error, source misattribution, or worst case, a fabricated citation).
4. **Acts on the verdict.**
   - `verified` → the key finding stands. Carry the `verified` verdict into the executive summary's sub-bullet.
   - `partially verified` → revise the key finding to match the source. Surface the revision in the Phase 5 delta if the change is material to the deliverable's conclusion. Carry the `partially verified — see note` verdict into the executive summary.
   - `not verified` → strike the key finding from the executive summary, or re-derive it from a different sourced support if one exists in the corpus. Surface the strike in the Phase 5 delta. Never silently retain a `not verified` key finding.
   - `unreachable` → see dead-link handling above.
5. **Records the verdict** in the source-validation log (working machinery — stays in the chat run sheet, never written into the deliverable file).

**Source-validation log shape (chat run sheet only).**

```
SOURCE-VALIDATION LOG

- Key finding: [statement]
  - Load-bearing URL: [URL]
  - Lane attribution (verbatim): "[what the lane said the source says]"
  - Source actual content (verbatim or close paraphrase): "[what the source actually says]"
  - Verdict: [verified / partially verified — see note / not verified / unreachable]
  - Action taken: [stand / revise + delta filed / strike / dead-link substitute or downgrade]
- [Repeat for each load-bearing source]
```

**When the source-validation pass exhausts the reduce step's budget.** If load-bearing sources outnumber the revisit budget (rare on a typical 3–7-key-finding deliverable; possible on larger ones), prioritize by impact on the overall conclusion: the URL most central to the conclusion goes first. Sources that cannot be reached within budget are marked `revisit deferred` in the log and surfaced in the Phase 5 delta as an explicit known gap — they do not silently inherit a `verified` verdict.

## Verify-and-converge stage brief

The verify-and-converge stage runs after the reduce step drafts key findings and runs the source-validation pass, but **before** the executive summary is finalized. The stage is **always-on** — every orchestrated deliverable runs through it. The workflow script spawns independent adversarial agents (optionally dispatched as `agentType: 'devils-advocate'` to pick up the bundled agent's role discipline); their verdicts feed the synthesis reduce step, which owns the final verdict that lands in the deliverable. The mode (adversarial or confirmatory) is chosen by the reduce step at spawn time using the sniff below.

### Mode sniff

The same sniff Phase 2-O uses for its Step 2B' adversarial-pass skip clause (see [landscape-research.md](landscape-research.md) § Step 2B' — Adversarial Pass, "Skip clause") applies here, adapted to the deliverable shape:

- **Adversarial mode** — the default. Use when the deliverable has an implicit thesis: it answers a question, takes a position, evaluates an option, recommends an action, draws a conclusion the audience could plausibly disagree with. Analytical, decision-support, evaluative, comparative, and recommendation-shaped deliverables all default to adversarial.
- **Confirmatory mode** — use when the deliverable is purely descriptive: an inventory, a fact-extract roll-up, a directory compilation, a roster, a chronology with no causal interpretation. There is no implicit thesis to attack; an adversarial pass would have to fabricate one.

The reduce step records the mode choice and a one-sentence reason in the stage brief and in the chat run sheet. When the choice is on the boundary, **default to adversarial** — over-running a confirmatory deliverable as adversarial wastes effort but does not corrupt the output (the verdict-ladder discipline will degrade gracefully to mostly-`confirmed` verdicts), while under-running an analytical deliverable as confirmatory lets a weak thesis ship without an adversarial test.

### Adversarial-mode brief

The brief embedded in each adversarial verify-stage `agent` call carries:

- The **key-findings list** with each finding's one-sentence statement and primary supporting URL(s).
- The **per-finding tentative conclusions** the reduce step drafted.
- The **explicit instruction**: "For each key finding, search for the strongest sourced evidence that the opposite of the finding is true. Return per-finding verdict: `confirmed` (no credible counter-evidence surfaced), `refuted by adversarial pass` (counter-evidence is strong enough that the original finding does not hold), or `unresolved — both views credible` (a sourced credible counterweight exists alongside the finding, and a sophisticated reader would weigh both). Cite every counter-evidence URL. Honor the topic-not-position discipline — identify where the opposite view is taken seriously; never adopt the opposite as your own conclusion."
- The **verdict-ladder discipline** verbatim (below).

### Verdict-ladder discipline (anti-mush guard)

`unresolved — both views credible` fires only when the adversarial evidence is from a **sourced, credible counterweight** that a sophisticated reader would actually update on — not whenever a contrary view merely exists. Speculative blog posts, single dissenting tweets, hypothetical objections without sourcing, and "some critics say…" framings without named critics or cited evidence do not qualify. The bar is: would the adversarial source survive a Tier-2+ citation in the source-trust hierarchy from [calibration-protocol.md](calibration-protocol.md)? If no, the verdict is `confirmed`, not `unresolved`.

Default verdict when adversarial searching surfaces nothing meeting that bar is **`confirmed`** — not `unresolved`. The failure mode this guards against is verdict-mush: every key finding hedges into "both sides," every conclusion becomes contingent, and the deliverable ships with no defensible position. A deliverable where every key finding is `unresolved` is a deliverable that has not drawn a conclusion.

### Confirmatory-mode brief

The brief embedded in each confirmatory verify-stage `agent` call carries:

- The **included entries list** with each entry's inclusion criteria and primary supporting URL(s).
- The **explicit instruction**: "Search for entries the fan-out lanes did not surface but that fit the deliverable's inclusion criteria. For each surfaced candidate, cite the URL and explain in one line why it meets the criteria. Independently, audit each included entry against its stated criteria — return per-entry verdict: `included correctly` (fits criteria), `included in error` (does not fit; explain), `boundary case` (debatable; surface the boundary and let the reduce step decide). Honor the topic-not-position discipline — surface findings, do not advocate for or against inclusions."
- A reminder that the confirmatory pass is **scoped** — it audits the surface inventory; it does not look for thesis-style counter-arguments because the deliverable does not advance a thesis.

### Verify-stage findings pack — output shape

Each verify-stage agent (whether an inlined adversarial brief or `agentType: 'devils-advocate'`) returns a structured findings pack. The reduce step then integrates it into the executive summary's verify-and-converge block.

**Adversarial mode return shape:**

```
VERIFY-AND-CONVERGE — ADVERSARIAL MODE

PER-FINDING VERDICTS
- Key finding 1: [confirmed / refuted / unresolved]
  - Adversarial evidence (if surfaced): [URL] — [one-line significance]
  - Why this verdict: [one-line reason — for `unresolved`, names the credibility ladder that puts both views above the bar]
- Key finding 2: [...]
- [...]

OVERALL VERDICT
- [holds / weakened / requires both-views framing]
- [one-line reason]
```

**Confirmatory mode return shape:**

```
VERIFY-AND-CONVERGE — CONFIRMATORY MODE

ENTRY AUDIT
- Entry 1: [included correctly / included in error / boundary case]
  - Why: [one-line reason; cite URL if a primary source resolved the call]
- [...]

MISSED ENTRIES (candidates the fan-out did not surface)
- Candidate A: [name] — [URL] — [one-line note on why it meets the criteria]
- [...]

OVERALL VERDICT
- [inventory complete and clean / inventory incomplete — N missed entries / inventory contaminated — N inclusion errors]
- [one-line reason]
```

The reduce step does not delegate **judgment** on adversarial verdicts to the verify-stage agents — they report `unresolved` when they cannot decide; the reduce step integrates the unresolved finding into the executive summary as `unresolved — both views credible` only after applying the verdict-ladder discipline above. The verify-stage agents supply evidence and a tentative classification; the reduce step owns the final verdict that lands in the deliverable.

## Authoring rules for the synthesis reduce step

These rules apply across the executive summary, the source-validation pass, the verify-and-converge integration, and the deliverable body.

- **Every claim cites a URL.** Reduce-step integrative reasoning is explicit ("Combining findings 1 and 3, …") rather than implicit. No claim floats sourceless in the deliverable body.
- **The deliverable is what gets written; the working machinery stays in chat.** The source-validation log, the verify-stage findings packs, the reduce step's working drafts, and any per-source verbatim quotes used during validation all live in the chat run sheet alongside the deliverable. The Phase 6B pre-write strip check from [`SKILL.md`](../SKILL.md) is extended to cover these new working artifacts; the written file contains only the deliverable body (executive summary + synthesized findings + supporting evidence + gaps + sources).
- **Confidence stamps are honest.** If a load-bearing source's revisit returned `partially verified — see note`, the confidence stamp on the key finding cannot be `High` — that is what `partially verified` means. If an adversarial verdict was `unresolved`, the per-finding conclusion either flips to a both-views framing or the key finding drops out of the overall conclusion's support chain; it cannot ride into the conclusion as if the verdict were `confirmed`.
- **The mode choice is recorded.** Adversarial vs. confirmatory is a one-sentence note in the executive summary's verify-and-converge block. Defaulting silently to one mode without naming the choice is opaque to the reader.
- **A short deliverable is a legitimate result.** If only two key findings survive the source-validation pass and the adversarial pass, the deliverable has two key findings. Padding to a target count using findings that did not survive validation is the failure mode the protocol is built to prevent.

## Worked example — executive summary for the Section 10 Acme SaaS competitive landscape

Populated against the Section 10 worked example deliverable, kept compact to demonstrate the template-to-instance mapping without overwhelming. (Sources here are illustrative; a real run would surface real URLs through the workflow's fan-out lanes.)

```markdown
## Executive Summary

Acme SaaS competes in mid-market workforce-management software ($20M–$200M ARR, North America). This brief assesses Acme's defensible competitive position, summarizes the most recent funding and M&A in the segment, and identifies the pricing dynamic that most shapes win rates.

### Key findings

- **Key finding 1 — Two competitors (Brightline, Tessera) have moved into Acme's ICP in the last 18 months via vertical-specific SKUs that undercut Acme on price by ~15%.**
  - Primary source: https://[brightline-pricing-page-and-tessera-launch-press-release]
  - Source-validation verdict: verified
  - Confidence: High
- **Key finding 2 — Acme's customer-segment overlap with Brightline is concentrated in the 100–500-FTE band; switching costs in this band are dominated by integration depth (HRIS + payroll + scheduling), not by training or contract length.**
  - Primary source: https://[g2-switching-cost-survey-and-customer-interview-set]
  - Source-validation verdict: partially verified — see note (the G2 survey is N=42 customers, not the full segment; switching-cost ranking holds but the sample caveat is now surfaced)
  - Confidence: Medium
- **Key finding 3 — Two of the four competitors named in the fan-out inventory have closed funding rounds in the last 12 months; one (Tessera) raised Series C at a valuation step-up that funds aggressive land-and-expand into Acme's accounts.**
  - Primary source: https://[tessera-series-c-announcement-and-pitchbook-deal-page]
  - Source-validation verdict: verified
  - Confidence: High

### Per-finding conclusions

- From key finding 1 → Price competition in the ICP has hardened from a soft variable to a hard one; pricing-model defense is now a near-term Acme priority.
- From key finding 2 → Acme's switching-cost moat is real but narrow; it protects the integration-deep installed base, not the broader segment, and it depreciates if Acme's integration breadth does not extend.
- From key finding 3 → Tessera is the immediate strategic threat (not Brightline) because the Series C funds 18–24 months of below-market pricing; defending against Tessera requires either matching the price posture or differentiating on integration depth.

### Overall conclusion

Acme's defensible position is narrowing, not collapsing. The most acute pressure is Tessera's funded ability to underprice in the 100–500-FTE band, where Acme's integration moat is real but does not generalize. A defensible 12-month plan combines integration-depth expansion (extending the moat into segments Tessera cannot match without comparable engineering investment) with selective price-posture defense in accounts where switching costs are below the integration-depth threshold. The brief does not recommend across-the-board price-matching.

### Verify-and-converge verdicts

- **Key finding 1:** confirmed — adversarial search surfaced one practitioner thread arguing Brightline's vertical SKUs are loss-leader pricing that will reset within 12 months; the thread is single-source and below the credibility bar, so the verdict stands.
- **Key finding 2:** unresolved — both views credible. Adversarial evidence (https://[forrester-mid-market-wfm-2026-q2]) argues integration depth is decaying as a moat across the segment as middleware platforms commoditize HRIS-payroll-scheduling glue; the source is Tier 4 (independent analyst) but credible and recent. Net effect: the per-finding conclusion is reframed to "moat is real but actively decaying; the integration-depth investment must outrun the commoditization curve."
- **Key finding 3:** confirmed — no credible counter-evidence to Tessera's Series C raise or its strategic implications surfaced.
- **Overall conclusion verdict:** holds — but the integration-depth recommendation is now load-bearing, given key finding 2's reframing.
- **Mode:** adversarial — the deliverable advances a recommendation on Acme's competitive position; a confirmatory mode would not surface the moat-decay argument that materially shifts the recommendation's framing.
```

## Cross-references

- [`SKILL.md`](../SKILL.md) Meta-Rule 13 — the orchestrator invariants this reference operationalizes for the deliverable.
- [dynamic-workflows.md](dynamic-workflows.md) §7 — the synthesis tasks (source-validation revisit and verify-and-converge integration are the final two).
- [dynamic-workflows.md](dynamic-workflows.md) §8 — the verify-and-converge stage whose verdicts feed the executive summary's verify-and-converge block.
- [dynamic-workflows.md](dynamic-workflows.md) §10 — the orchestrator CLAUDE.md template, whose Workflow plan / Synthesis / Deliverable sections embed the slots this reference fills.
- [dynamic-workflows.md](dynamic-workflows.md) §11 — failure mode catalog, with rows for the failure modes this reference is built to prevent.
- [qa-checklist.md](qa-checklist.md) — the QA gates the synthesis reduce step's deliverable must pass.
- [calibration-protocol.md](calibration-protocol.md) — the source-trust hierarchy referenced by the verdict-ladder discipline.
- [landscape-research.md](landscape-research.md) § Step 2B' Skip clause — the sniff reused for adversarial / confirmatory mode selection.

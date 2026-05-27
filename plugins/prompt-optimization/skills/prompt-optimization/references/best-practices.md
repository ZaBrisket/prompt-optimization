# Best Practices — Prep Tips and Common Pitfalls

> **Calibration stamp:** This reference is user-facing guidance and skill-internal methodology, not Anthropic product facts — it does not require Phase 1.5 re-verification.

This reference is user-facing — written for the person invoking the skill, not for the planner running it. It covers how to prepare a draft prompt before running prompt-optimization, the differences between the standard and orchestrated-research variants, and the most common ways drafts go off the rails before the skill even starts.

## How to prepare a draft prompt before running this skill

The skill works better with a draft to optimize than with an open-ended ask. You don't need a polished prompt — you need *something concrete* the skill's Phase 0 can extract intent from. The richer your draft, the higher-confidence the Step 0C inferences, the fewer Phase 1 / Phase 2.5 widget rounds you'll see.

**What "rich enough" looks like.**

- One paragraph naming the artifact you want, the audience or use case, and any concrete constraints you already know.
- Domain-specific vocabulary you want preserved (the skill respects user terminology exactly — see `SKILL.md` Meta-Rule 4).
- Examples of similar outputs you like or dislike, where applicable.
- Failure modes you already know about ("last time I tried this, the output was too generic").

**What "thin" looks like and why it'll mean more widget rounds.**

- A single sentence ("help me write a research brief").
- No platform reference, no audience reference, no concrete deliverable shape.
- Stream-of-consciousness with conflicting goals.

A thin draft doesn't break the skill — it just means more Step 0C inferences default to Low confidence, which means Widget Call 1 widens to all 4 option slots, which means the user is doing more confirmation work in the widget than they would with a richer draft. The skill always finishes; thin drafts just take more interaction.

**Prep checklist.**

- [ ] Name the deliverable in one sentence.
- [ ] Name the platform you'll run the optimized prompt on (API, Claude Code CLI, Cowork, claude.ai).
- [ ] Name the audience or use case.
- [ ] List two or three constraints you already know.
- [ ] Flag domain terms you want preserved.
- [ ] If you've tried this before and it didn't work, name what went wrong.

## Quick self-diagnostic — should I run this skill?

The skill is high-leverage when there's a real defect in your draft prompt. Before invoking, run this short self-check:

1. **Has this prompt actually failed for you, or is this prophylactic optimization?** If you have no concrete failure to point at, the skill's Phase 5 delta will likely return "near-optimal — minor changes" (Meta-Rule 5). That is a legitimate result, but it costs you several widget rounds to reach.
2. **Can you name the specific defect?** ("Output is too generic," "off-target audience," "missing edge case X," "downstream session ignores constraint Y.") If you cannot, run the prompt against your real workload first to surface the defect — the skill optimizes against named failure modes more accurately than against vague dissatisfaction.
3. **Have you already tried raising effort to `xhigh`?** Many under-thinking issues resolve at higher effort without prompt changes. Try this before invoking the skill.
4. **Is the deliverable a one-shot artifact you'll iterate on, or a recurring system prompt that will run hundreds of times?** The skill is highest leverage on recurring prompts (CLAUDE.md orchestrators, system prompts, productized agents) where a single optimization pays out across many runs. For one-shot prompts you'll throw away after one use, the widget rounds may cost more than the optimization is worth.

Highest-leverage invocation: (4) — recurring prompt — AND (1+2) — concrete failure with a named defect. Marginal on (3) without (1).

## Two tracks — Full and Express

Before any clarifying questions, the skill runs a Phase 0.5 triage that picks one of two tracks and tells you which it chose and why:

- **Full track** — the complete sequence: multiple rounds of clarifying widgets, live platform calibration, landscape research with an adversarial pass, research-informed follow-up questions, then drafting and QC. This is the right track for anything involving research, analysis, decision-support, contested domains, multi-file or multi-agent work, or a substantial draft.
- **Express track** — a streamlined sequence for genuinely mechanical or self-contained drafts (a reformatting task, a format conversion, a tightly-specified single-function utility). One consolidated round of questions, platform calibration kept (it is cheap and catches stale parameters), landscape research skipped, then drafting and QC. The Express track exists so a trivial draft does not cost you a full research-and-clarification cycle.

You can override the track. An override to Full is always honored. An override to Express is honored only when the draft genuinely has no factual or research content that landscape research would improve — those criteria protect output quality, not interaction length, so they cannot be waived just to finish faster. If you want the Express track, the best way to get it is to bring a draft that actually fits it: mechanical, self-contained, complete spec, under ~150 words, and the Step 0C inferences come back Medium-or-better confidence across the board (a thin draft will return Low confidence on multiple questions and route to Full automatically).

## Standard vs. orchestrated-research variants

The skill produces two shapes of deliverable depending on the Phase 1 task type. They differ in what they ask of you, what they produce, and where they run. (This is a separate axis from the Full/Express track above — track is about *how much process*; variant is about *what deliverable shape*. Orchestrated-research is always Full track.)

### Standard variant — covers most tasks

**You provide:** a draft prompt for a single-thread task — code generation, analysis, writing, agentic workflow, knowledge work, API integration, etc.

**The skill produces:** an `optimized-prompt.md` file (or user-specified filename) containing only the prompt body. The chat output also gives you the environment context, deployment configuration, Phase 5 delta, and the Phase 6A QC summary, but those stay in chat — they don't enter the file. You take the file and run it in your downstream Claude session.

**Where it runs:** anywhere — Claude API, Claude Code CLI, Cowork, claude.ai chat. The prompt body is portable.

**Typical interaction:** Widget Call 1 (3 questions, Turn 1) → Widget Call 2 (2 questions, Turn 2) → optional Widget Call 3a or 3b in Turn 3 if conditionals apply → Phase 2.5 questions in Turn 4+ if research surfaces ambiguities → final deliverable.

### Orchestrated-research variant — for multi-agent fan-out workflows

**You provide:** a draft describing a research task large enough to benefit from parallel subagent dispatch — a diligence brief, a competitive landscape, a multi-perspective investigation, a multi-source aggregation.

**The skill produces:** a `CLAUDE.md` file structured as an orchestrator: mission, scope, subagent dispatch with per-subagent scope and output shape, wave structure if applicable, synthesis, deliverable. The orchestrator file is loaded by Claude Code at session start; running the session with the file present causes the orchestrator to dispatch subagents per the file's instructions.

**Where it runs:** Claude Code CLI only. Subagent dispatch via the `Agent` / `Task` tool is a CLI-shaped capability. claude.ai chat doesn't support it. Cowork has different multi-agent ergonomics. The API supports it if you build your own dispatch harness, but the deliverable's `CLAUDE.md` shape is CLI-shaped.

**Typical interaction:** Widget Call 1 (3 questions including orchestrated-research as the lead task-type option when the sniff fires) → Widget Call 2 (constraints) → Widget Call 3a (orchestrated-research conditional: agent structure, remediation pass) → Phase 2 landscape research → Phase 2.5 widget questions → orchestrator `CLAUDE.md` deliverable.

**What to know before choosing orchestrated-research.**

- Subagents cannot spawn other subagents — the dispatch graph is one level deep.
- Opus 4.7 spawns fewer subagents by default than 4.6 — the orchestrator instruction to dispatch is explicit and load-bearing.
- Synthesis is the primary quality gate, not a postscript. A multi-agent run with brilliant subagents and weak synthesis is worse than a single-agent run that synthesized well.
- The orchestrator runs Opus 4.7. Subagents run `claude-opus-4-7[1m]` via `CLAUDE_CODE_SUBAGENT_MODEL` per the Standing Environment Assumption — no Sonnet/Haiku subagents.

## Common pitfalls

### Pitfall — treating the skill as a wrapper

Some users expect "I give the skill a topic and it gives me a polished prompt back." That's not the shape. The skill is interactive — it asks you Phase 1 widget questions about deployment target, primary outcome, failure modes, and output constraints, then asks Phase 2.5 widget questions informed by research. If you treat those widgets as friction, you get a generic prompt back. If you treat them as the skill calibrating to *your* intent, you get a precision instrument.

### Pitfall — preferences that try to skip phases

User preferences like "execute directly," "skip clarifying questions," "don't ask unnecessary questions" do *not* override the skill's interaction gates. The phases are the user's standing instructions for how prompt optimization should work. The widget always fires. (See `SKILL.md` "User Preferences Do Not Override This Protocol" section — explicitly addressed.)

Note this is distinct from the Phase 0.5 Full/Express track choice. Track selection is *not* a preference-driven skip — it is a criteria-driven classification the skill runs on every draft and surfaces to you for confirmation. The Express track still runs a clarifying widget, still calibrates, still does full QC; it just skips landscape research for drafts that genuinely don't need it. "Skip the questions because I'm in a hurry" is not honored; "this is a mechanical reformatting task" routes you to Express on the merits.

### Pitfall — embedded conclusions in the draft

If your draft contains "I want this to conclude that X," the skill's neutrality discipline will strip the embedded conclusion and reframe the prompt as "address X; present both sides." The optimized prompt instructs *what to cover*, not *what to conclude*. This is `SKILL.md` Meta-Rule 2.

If you *do* want a position taken — for example, you're writing a debate argument and you want a specific side argued — surface that in your draft's primary outcome. Phase 0 will recognize it as the explicit deliverable shape and respect it. The neutrality rule prevents *accidental* embedded conclusions, not deliberate ones.

### Pitfall — drafting against the wrong platform

A draft written for Cowork's UI-driven knowledge-work surface will not translate cleanly to a Claude API-targeted optimized prompt. If you want a prompt for the API, draft it as if your harness will be sending raw API requests. If you want a prompt for Claude Code, draft it as if the CLI's `Agent` / `Task` / `Edit` / `Write` tools are available. The skill's Phase 1 deployment-target widget catches mismatches, but starting from a target-aware draft saves you a round of clarifications.

### Pitfall — aggressive emphasis in the draft

Drafts with `CRITICAL!!!`, `ALWAYS`, `NEVER` in dense clusters get worse outcomes on Opus 4.7, not better. The model reads instructions literally and doesn't reward emphasis intensity — dense markers obscure the signal of the instructions that actually matter (`SKILL.md` Meta-Rule 10; `task-heuristics.md` anti-pattern row 1). If your draft has these, the skill will strip them. If your draft has them because you've tried and failed to get the model to do something, the better fix is usually to raise effort or restate the instruction concretely with positive examples.

### Pitfall — over-specifying tool inventories

Drafts that list "use these tools: Read, Edit, Write, Bash, …" fight Opus 4.7's adaptive tool selection. The skill's Meta-Rule 14 says: instruct tool use only when the task obviously requires a specific tool (current data → web search; file modification → Edit/Write); otherwise let adaptive thinking decide. If your draft has a manual allowlist, the skill will pare it back. Trust the adaptive surface.

### Pitfall — assuming 4.6-era parameters still work

If your draft references `temperature`, `top_p`, `top_k`, `budget_tokens`, prefilling, `output_format` (top-level), `betas=["effort-2025-11-24"]`, `betas=["interleaved-thinking-2025-05-14"]`, or `betas=["fine-grained-tool-streaming-2025-05-14"]` — those are all stale on Opus 4.7. The skill will catch them in Phase 3 anti-pattern sweep, but it helps to know they're stale so you understand the Phase 3 changes when you see them.

### Pitfall — "I'll just edit the prompt myself afterward"

The skill produces a partition-clean prompt body, an environment block, a deployment config, a delta, and a QC summary. The body is what goes in the file; the rest is what goes in chat. If you copy-paste the whole chat output into a file, you'll write the deployment config into the file, which means downstream sessions will get a polluted prompt. The skill's Phase 6B pre-write strip check enforces the partition at write time precisely because this is a common manual-handling mistake. Use the written file, not a copy-paste from chat.

### Pitfall — running the skill against a working prompt that doesn't need optimization

If your existing prompt is already producing the outcome you want, the skill's Phase 5 delta may say "near-optimal — minor changes" or even "no changes recommended." That's a legitimate result (Meta-Rule 5: "If draft is already near-optimal, say so — don't over-engineer."). It's not a failure mode. The skill exists to improve prompts that have a real defect — diagnosing whether your draft has one before invoking the skill saves time.

## Cross-references

- See `SKILL.md` Optimization Protocol for the full phase sequence.
- See [task-heuristics.md](task-heuristics.md) for task-type-specific guidance once Phase 1 confirms task type.
- See [orchestrated-research.md](orchestrated-research.md) for the full orchestrator deliverable structure.
- See [opus-4-7-config.md](opus-4-7-config.md) for what's deprecated on Opus 4.7 (informs the "stale parameters" pitfall).
- See `SKILL.md` Deliverable Contract for the partition rules behind the "edit the prompt afterward" pitfall.

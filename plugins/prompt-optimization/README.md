# prompt-optimization (plugin)

Transform a draft prompt, system message, `CLAUDE.md`, or multi-agent orchestrator into production-grade instructions for Claude Opus 4.8. Calibrated against live Anthropic documentation on every run.

## Components

| Type | Name | Purpose |
|------|------|---------|
| Skill | `prompt-optimization` | The phased optimization protocol. Auto-triggers on prompt-improvement intent, or invoke explicitly with `/prompt-optimization`. |
| Agent | `landscape-research` | Read-only research worker dispatched as a workflow agent during orchestrated Phase 2 (mode 2-O), with a parallel-subagent fallback when the workflow runtime is unavailable. |
| Agent | `devils-advocate` | Read-only adversarial / confirmatory worker dispatched by the dynamic workflow's verify-and-converge stage of every orchestrated-research deliverable. |

## How to invoke

- **Drafts in text:** Paste the draft into chat with optimization intent ("optimize this prompt: …", "harden this CLAUDE.md", "refine this system message"). The skill auto-triggers.
- **Drafts in a file:** Type `/prompt-optimization`; when the skill asks for the draft, paste the path.
- **Start cold:** Type `/prompt-optimization` and let the skill guide you through Phase 0.

## How it works

The skill runs a sequential state machine. Phase 0.5 triages each draft onto a **Full** or **Express** track:

- **Full track** — intent extraction → widget clarification → live calibration → landscape research → research-informed clarification → draft analysis → prompt construction → delta → QC + file write.
- **Express track** — for mechanical, self-contained drafts: one consolidated clarification round, mandatory live calibration, then construction → delta → QC + file write (skips landscape research).

Live calibration (Phase 1.5) is mandatory on every run — it catches model, API, and best-practice changes since the skill was last edited, so guidance never goes stale. It also re-verifies the Opus 4.8 system-card behavioral findings the Phase 3 anti-pattern sweep relies on (the §2.3.3 long-horizon failure modes, eval/grader-awareness posture, refusal calibration, and agentic-safety regressions on browser/computer use).

### Two Phase 2 execution modes

Landscape research (Full track) runs in one of two modes that converge on the same synthesis and neutrality / coverage-bias gates:

- **2-S (single-threaded)** — the default, and the only mode when the Agent/Task tool is unavailable. One pass, capped at 12 searches.
- **2-O (orchestrated fan-out)** — when the draft is multi-sub-domain, multi-stakeholder, or contested, the skill fans out via a Claude Code **dynamic workflow**: one `landscape-research` workflow agent per lane (plus a dedicated adversarial lane), each searching deeply in its own isolated context, then synthesizes the aggregate. Bounded by ~6 lanes (synthesis bandwidth), per-lane budgets, and one remediation pass. Falls back to parallel subagents via the Agent/Task tool when the workflow runtime is unavailable.

The chosen mode and its rationale are reported in the run sheet and the delta analysis.

## Deliverable contract

The written file contains **only** the optimized prompt body. Environment assumptions, deployment configuration, delta analysis, and the QC summary stay in the chat run sheet — never in the file. The skill enforces this with a pre-write strip check.

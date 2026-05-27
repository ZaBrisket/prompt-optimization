---
name: optimize-prompt
description: Optimize a draft prompt, system message, CLAUDE.md, or multi-agent orchestrator into production-grade instructions for Claude Opus 4.7. Runs the full phased protocol with live documentation calibration and, where eligible, orchestrated multi-agent landscape research. Not for one-off content writing, code generation, or answering a prompt's own question rather than improving the prompt itself.
argument-hint: <draft prompt text, a file path to a draft, or a description of what to optimize>
---

# /optimize-prompt

Treat this as an explicit request to optimize, harden, and calibrate a prompt using the **prompt-optimization** skill. Invoke that skill and follow its protocol in full.

## Input

$ARGUMENTS

## Handling

- If the input above is a **file path**, read that file and treat its contents as the draft to optimize.
- If the input above is **prompt text or a description**, treat it as the draft.
- If the input above is **empty**, ask the user to paste the draft prompt or provide a file path before proceeding. Do not invent a draft.

## Protocol guarantees

Run the skill's published protocol without shortcuts:

- Do **not** skip the skill's interaction gates. The phased widget questions are deliberate — a general "execute directly / don't ask questions" preference does not override them (see the skill's "User Preferences Do Not Override This Protocol").
- Phase 1.5 live calibration against Anthropic documentation is mandatory on every run.
- Let Phase 0.5 select Full vs. Express track on its published criteria, and surface the decision to the user.
- For Full-track factual / research / decision-support drafts, let Phase 2 choose its execution mode (single-threaded vs. orchestrated fan-out) on the published criteria, and report which mode ran.
- Honor the Deliverable Contract: the written file contains only the optimized prompt body; the environment block, deployment config, delta analysis, and QC summary stay in the chat run sheet.

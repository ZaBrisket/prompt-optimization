# prompt-optimization

A single-plugin [Claude Code](https://code.claude.com/docs/en/overview) marketplace hosting the **prompt-optimization** plugin: a phased workflow that turns a draft prompt, system message, `CLAUDE.md`, or multi-agent orchestrator into production-grade instructions for Claude Opus 4.8.

## Install

### Claude Code (CLI)

```bash
claude plugin marketplace add ZaBrisket/prompt-optimization
claude plugin install prompt-optimization@prompt-optimization
```

Or, from within an interactive Claude Code session:

```
/plugin marketplace add ZaBrisket/prompt-optimization
/plugin install prompt-optimization@prompt-optimization
```

The first command registers this repo as a marketplace; the second installs the plugin from it (`<plugin>@<marketplace>`). Restart Claude Code if prompted so the new skill and agents load.

> Dynamic-workflow orchestration is a research preview and requires Claude Code v2.1.154 or later.

### Cowork (Desktop)

1. In the Cowork desktop app, open the **Customize** menu and add this repo (`ZaBrisket/prompt-optimization`) as a marketplace.
2. Install **prompt-optimization** from the marketplace.

### Local development

```bash
claude plugin marketplace add /path/to/this/repo
claude plugin install prompt-optimization@prompt-optimization
```

## What you get

- **Skill — `prompt-optimization`** — auto-triggers when you provide a draft prompt to improve, ask to optimize/harden/refine a prompt, or describe a multi-agent research workflow. Runs a sequential protocol: intent extraction, complexity triage, widget-based clarification, mandatory live calibration against Anthropic documentation, landscape research, draft analysis, prompt construction, delta analysis, and a final QC pass with a clean file write. Invoke explicitly with `/prompt-optimization` if the auto-trigger doesn't fire.
- **Agent — `landscape-research`** — a read-only research worker dispatched as part of the orchestrated Phase 2 (mode 2-O) dynamic workflow — one lane per sub-domain or query-taxonomy cluster — with a parallel-subagent fallback when the workflow runtime is unavailable; results synthesize centrally.
- **Agent — `devils-advocate`** — a read-only adversarial / confirmatory worker dispatched by the verify-and-converge stage of every orchestrated-research deliverable: adversarial mode for thesis-advancing deliverables, confirmatory mode for purely descriptive inventory / fact-extract deliverables.

## Orchestrated landscape research (mode 2-O)

When the draft is multi-sub-domain, multi-stakeholder, or contested, Phase 2 fans out via a Claude Code **dynamic workflow**: one `landscape-research` lane per sub-domain or query-taxonomy cluster, plus a dedicated adversarial lane for the verify-and-converge stage, each searching deeply in its own isolated context. The orchestrator's reduce step collects the lane returns and runs synthesis and the neutrality / coverage-bias gates on the aggregate corpus. Fan-out is bounded by ~6 lanes (synthesis bandwidth), the runtime caps (≤16 concurrent / ≤1,000 total agents per run), per-lane search budgets, and at most one remediation pass. When the dynamic-workflow runtime is unavailable, Phase 2 falls back to parallel subagents via the Agent/Task tool (or single-threaded execution on claude.ai chat / bare API) automatically.

The orchestrated-research deliverable the plugin produces is itself a `CLAUDE.md` that orchestrates a dynamic workflow.

## Repository layout

```
.
├── README.md                                # this file
├── .claude-plugin/marketplace.json          # marketplace definition
└── plugins/
    └── prompt-optimization/
        ├── .claude-plugin/plugin.json        # plugin manifest
        ├── skills/prompt-optimization/       # SKILL.md + references/
        ├── agents/landscape-research.md      # landscape research worker
        ├── agents/devils-advocate.md         # devil's-advocate / confirmatory worker
        └── README.md
```

## Author

Mac Zabriskie

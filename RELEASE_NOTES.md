# AgentFlow v1.0.0

A single-file, drag-and-drop tool for diagramming workflows and process maps — then
handing them to Claude (or any agent) as rich, structured context. AgentFlow reads and
writes native **JSON**, **Markdown**, and **Mermaid**, and uses the workflow's **graph
structure** to validate flows and slice them into focused, per-node context for reliable
agent execution.

No build step, no dependencies, no account — `agentflow.html` is one self-contained file.
Download it and double-click.

## Highlights

- **Visual editor** — drag-and-drop nodes, connect ports, label branches, pan/zoom,
  Shift-click multi-select, and group-move.
- **Seven context-rich node types** — Objective, Step, Subagent, Decision, Gate, Context,
  and Deliverable — built around what makes an agent reliable: intent, inputs/outputs,
  acceptance criteria, human gates, and delegation.
- **Three formats, in and out:**
  - **JSON** — the canonical, lossless source (coordinates optional; the app auto-lays-out).
  - **Markdown** — the ordered, branch- and gate-aware spec an agent executes (lossless round-trip).
  - **Mermaid** — export a diagram that renders natively on GitHub; import one as a structure on-ramp.
- **Graph validation** — a live `✓ Valid / ⚠ N issues` chip and a click-to-jump panel that
  catches dead ends, unlabeled or under-branched decisions, orphan/unreachable nodes,
  gates with no exit, and edges that violate start/end rules.
- **Context packets** — hand an agent exactly the slice it needs for one node: the global
  frame plus that node's upstream and downstream interfaces. Copy a single node's packet,
  or export all packets in execution order for handing independent branches to subagents.
- **Reusable library** — built-in blocks (Design Review loop, Discovery → Wireframe,
  Research spike, Medallion pipeline, Data quality gate) plus save-your-own selections.
- **Agent-authorable** — a documented authoring contract (`AGENTS.md`), tolerant JSON
  ingest, and an in-app **Copy Agent Prompt** so any Claude session can build a workflow
  without opening the app.

## Getting started

- **Just run it:** download `agentflow.html` from the assets below and double-click it.
- **Or clone** the repo and open the same file — no install required.
- **For agents:** see `AGENTS.md` for the authoring contract; `CLAUDE.md` auto-loads it for
  Claude Code sessions in this folder.

## Included example

A worked Databricks financial-close pipeline (`financial-close-pipeline.json`) and a
ready-to-copy `handoff/` bundle showing how to drop a workflow into a working project.

## License

MIT.

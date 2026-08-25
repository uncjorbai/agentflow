# AgentFlow

A single-file, Visio-style visual workflow editor for **guiding Claude through design and
build processes**. You drag out a process, each node carries the context Claude needs to
execute that step, and the tool compiles the diagram into a spec an agent can follow.

No build step, no dependencies — [`agentflow.html`](agentflow.html) is one self-contained
file. Double-click to open.

## Why

A diagram is only useful to an agent if it carries real execution context. AgentFlow's
node types are built around what makes Claude reliable — intent, inputs/outputs,
acceptance criteria, human gates, and delegation — not generic flowchart boxes. Because
the workflow is a graph, the app can validate it, render it, and slice it into focused
per-node briefs. It compiles to an ordered, branch-and-gate-aware Markdown spec any Claude
session can execute.

## Using the app

- **Drag** a shape from the left palette onto the canvas.
- **Click** a node to edit its context fields on the right.
- **Drag** a node's right-edge dot onto another node to connect them.
- **Click** an arrow to label a branch (Yes / No / on failure).
- **Shift-click** nodes to multi-select; drag to move them as a group.
- Scroll to zoom, drag empty space to pan, `Delete` to remove the selection.

### Menus

- **File** — New · Open `.json` · Open `.md` · Open `.mmd`
- **Export** — Export `.json` · Export `.md` · Export `.mmd` · Copy for Claude (Markdown) ·
  Copy Agent Prompt · Copy node packet · Export all packets
- **Library** — insert built-in blocks or save your own selection as a reusable block.
- A live **validation chip** in the header (`✓ Valid` / `⚠ N issues`) — click it for a
  panel that jumps to each problem node.

### Node types

| type | meaning |
|------|---------|
| Objective (start) | Workflow goal + global context & constraints |
| Step | A unit of work Claude performs (intent, inputs, outputs, done-when) |
| Subagent | Work delegated to a specialized agent |
| Decision | A branch; outgoing edges are the choices |
| Gate | A human-approval checkpoint |
| Context | Shared reference material |
| Deliverable (end) | The final output + success criteria |

## Formats

| Format | Export | Import | Fidelity | Use it for |
|--------|:------:|:------:|----------|------------|
| **JSON** | ✓ | ✓ | Lossless (canonical) | Save/load; the source of truth; agent authoring |
| **Markdown** | ✓ | ✓ | Lossless round-trip | The spec Claude executes; human reading |
| **Mermaid** | ✓ | ✓* | Lossy — skeleton only | Rendering a diagram in GitHub/docs; sketch on-ramp |

\* Mermaid import reconstructs structure only — titles, types (from shape), edges, and
branch labels. Context fields come in blank to fill. Our own Mermaid export round-trips
its skeleton cleanly; arbitrary third-party Mermaid is parsed best-effort.

## Validation

AgentFlow understands its own graph and flags issues before you hand a workflow off:
edges into a start or out of an end (errors), dead-end nodes, decisions with too few or
unlabeled branches, gates with no exit, orphan or unreachable nodes, missing
Objective/Deliverable, and untitled nodes. The header chip stays live as you edit.

## Context packets

Because the workflow is a graph, AgentFlow can hand an agent exactly the slice it needs for
one node instead of the whole spec. A **context packet** is a standalone brief: the global
frame (objective + guardrails), the node's own detail, and its **upstream** and
**downstream** interfaces (what it receives, what it must produce) derived from the graph
edges. Select a node and **Copy context packet**, or **Export all packets** for the whole
workflow in execution order — a natural fit for handing independent branches to separate
subagents.

## For agents

Any Claude session can author a workflow without opening the app. See
[`AGENTS.md`](AGENTS.md) for the full authoring contract (JSON schema, node fields,
paper-sketch → JSON examples). [`CLAUDE.md`](CLAUDE.md) auto-loads that contract for
Claude Code sessions opened in this folder. In short: emit a `.json` (coordinates
optional — the app auto-lays-out) and open it with **File → Open .json**.

The in-app **Export → Copy Agent Prompt** puts a condensed version of the contract on
your clipboard for pasting into any chat.

## Handing a workflow to a working project

Export the workflow as Markdown and drop it into the target project. The
[`handoff/`](handoff/) folder is a worked example (a Databricks financial-close pipeline):
copy its `CLAUDE.md` + `.md` into a project root and any Claude Code session there will
follow the plan. The `.json` is the editable source; the `.md` is what Claude executes.

## Files

- `agentflow.html` — the app (self-contained; double-click to open)
- `AGENTS.md` — authoring contract for agents
- `CLAUDE.md` — auto-loads the contract for Claude Code
- `financial-close-pipeline.json` — example workflow (editable source)
- `handoff/` — example handoff bundle (spec + CLAUDE.md pointer)
- `LICENSE` — MIT

## License

MIT — see [`LICENSE`](LICENSE).

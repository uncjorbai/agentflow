# AgentFlow

A single-file, Visio-style visual workflow editor for **guiding Claude Code through
design and build processes**. You drag out a process, each node carries the context
Claude needs to execute that step, and the tool exports a spec that Claude follows.

No build step, no dependencies — [`agentflow.html`](agentflow.html) is one self-contained
file. Double-click to open.

## Why

A diagram is only useful to an agent if it carries real execution context. AgentFlow's
node types are built around what makes Claude reliable — intent, inputs/outputs,
acceptance criteria, human gates, and delegation — not generic flowchart boxes. The
diagram compiles to an ordered, branch-and-gate-aware Markdown spec that any Claude
session can execute.

## Using the app

- **Drag** a shape from the left palette onto the canvas.
- **Click** a node to edit its context fields on the right.
- **Drag** a node's right-edge dot onto another node to connect them.
- **Click** an arrow to label a branch (Yes / No / on failure).
- **File** menu: New · Open .json · Open .md. **Export** menu: Export .json · Export .md ·
  Copy for Claude · Copy Agent Prompt.
- **Library**: insert built-in blocks (Design Review loop, Discovery → Wireframe, Research
  spike) or save your own selections as reusable blocks.

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

- `agentflow.html` — the app
- `AGENTS.md` — authoring contract for agents
- `CLAUDE.md` — auto-loads the contract for Claude Code
- `financial-close-pipeline.json` — example workflow (editable source)
- `handoff/` — example handoff bundle (spec + CLAUDE.md pointer)

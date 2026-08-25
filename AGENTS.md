# AgentFlow — Authoring Contract for Agents

This file is the **stable contract** any agent (any session, any model) follows to
create or edit an AgentFlow workflow. If a user hands you a diagram — a photo of a
whiteboard, a paper sketch, a description in prose — your job is to translate it into
a file AgentFlow can open. Read this whole file first; do not reverse-engineer the
format from `agentflow.html`.

AgentFlow is a visual "context builder": boxes are steps of a process, arrows are the
order, and each box carries the context Claude needs to execute that step. The point
of a workflow is the **exported spec**, so richer node fields = a better result.

---

## The golden path: emit JSON

**Produce a `.json` file. The user opens it with the "Open .json" button.** That is the
whole loop. Prefer JSON over Markdown — it is unambiguous and lossless.

You do **not** compute coordinates. If nodes omit `x`/`y`, AgentFlow auto-lays-out the
graph left-to-right by flow depth. Only include coordinates if the user asks for a
specific arrangement.

### Minimal shape

```json
{
  "name": "Short workflow name",
  "nodes": [
    { "id": "a", "type": "start", "data": { "title": "Objective", "objective": "..." } },
    { "id": "b", "type": "step",  "data": { "title": "Do the thing", "instructions": "..." } }
  ],
  "edges": [
    { "from": "a", "to": "b", "label": "" }
  ]
}
```

### Rules that must hold

- `nodes[].id` — any unique string. Semantic ids (`"research"`, `"review"`) are fine and encouraged.
- `nodes[].type` — one of the seven types below. Anything unknown is coerced to `step`.
- `edges[].from` / `edges[].to` — must match node ids. Edges referencing a missing id are silently dropped, so check them.
- `edges[].label` — optional; **required in spirit for `decision` branches** (see below).
- `x` / `y` — omit them. Auto-layout handles placement.
- Every field in `data` is optional except you should always give a `title`. Fill as many fields as the source supports — empty fields are simply omitted from the export.

---

## Node types and their `data` fields

| type       | shape it draws | meaning | `data` fields (all strings) |
|------------|----------------|---------|------------------------------|
| `start`    | rounded, green | The workflow's goal + global context. Put ONE per workflow at the entry. | `title`, `objective`, `context`, `constraints` |
| `step`     | rectangle, blue | A unit of work Claude performs. | `title`, `goal`, `instructions`, `inputs`, `outputs`, `references`, `acceptance` |
| `subagent` | rectangle, purple | Work delegated to a specialized agent. | `title`, `agent`, `goal`, `instructions`, `inputs`, `outputs`, `acceptance` |
| `decision` | diamond, amber | A branch. Outgoing edges are the choices. | `title` (phrase as a question), `criteria` |
| `gate`     | rounded, red | A human-approval checkpoint. Flow pauses here. | `title`, `review`, `criteria` |
| `context`  | note, gray | Shared reference material, not a step. | `title`, `content` |
| `end`      | rounded, teal | The final deliverable. Has no outgoing edges. | `title`, `deliverable`, `success` |

Field meanings that matter for quality:

- **`goal`** = *intent / why this step exists.* This is the highest-value field — it lets Claude adapt when reality differs from the plan. Always try to capture it.
- **`inputs` / `outputs`** = the artifacts a step consumes and produces. Filling these makes the chain legible (step B knows it consumes step A's output). This is the single biggest lever for reliable multi-step execution.
- **`acceptance`** = a verifiable "done when…" so Claude can self-check.
- **`constraints`** (on `start`) = global guardrails: tech, style, scope, what must never happen.
- **`agent`** (on `subagent`) = the agent type to delegate to, e.g. `Explore`, `general-purpose`, or a named custom subagent.

---

## Edges: order, branches, gates

- A plain edge `{from, to}` means "then." Give it no label.
- A **`decision`** node should have 2+ outgoing edges, each with a `label` naming the choice (`"Yes"`, `"No"`, `"on failure"`). The export renders these as explicit "if X → go to N" routing.
- A **`gate`** typically has one outgoing edge labeled `"approved"` (or similar), meaning "continue once a human approves."
- **Loops are allowed.** A revise-and-resubmit cycle is just an edge pointing back to an earlier node (e.g. `decision --No--> subagent --resubmit--> earlier step`).
- Do **not** draw an edge into a `start` or out of an `end`.

---

## Worked example — a paper sketch → JSON

Suppose the user photographs this sketch:

```
[Goal: ship onboarding v2] -> (Audit current flow) -> <Gaps found?>
        <Gaps found?> --yes--> (Redesign screens) -> [/Human review/] -> (Build) -> ((Done))
        <Gaps found?> --no---> ((Done))
```

Translate shapes: a labeled goal box → `start`; rectangles → `step`; the `<...>`
diamond → `decision`; `/.../` → `gate`; `((...))` → `end`. Then:

```json
{
  "name": "Onboarding v2",
  "nodes": [
    { "id": "goal",   "type": "start",    "data": { "title": "Ship onboarding v2", "objective": "Ship a better first-run experience", "constraints": "No new dependencies; match design system" } },
    { "id": "audit",  "type": "step",     "data": { "title": "Audit current flow", "goal": "Find where users drop off", "instructions": "Walk the existing onboarding and log friction", "outputs": "Friction list" } },
    { "id": "gaps",   "type": "decision", "data": { "title": "Gaps found?", "criteria": "Are there material problems worth fixing?" } },
    { "id": "design", "type": "step",     "data": { "title": "Redesign screens", "goal": "Remove the friction found", "inputs": "Friction list", "outputs": "New screen designs", "acceptance": "Every logged friction point is addressed" } },
    { "id": "review", "type": "gate",     "data": { "title": "Human review", "review": "The redesigned screens", "criteria": "Stakeholder approves the direction" } },
    { "id": "build",  "type": "step",     "data": { "title": "Build", "inputs": "New screen designs", "outputs": "Shipped feature", "acceptance": "Matches design; tests pass" } },
    { "id": "done",   "type": "end",      "data": { "title": "Done", "deliverable": "Onboarding v2 behind a flag", "success": "Drop-off measurably reduced" } }
  ],
  "edges": [
    { "from": "goal",   "to": "audit" },
    { "from": "audit",  "to": "gaps" },
    { "from": "gaps",   "to": "design", "label": "yes" },
    { "from": "gaps",   "to": "done",   "label": "no" },
    { "from": "design", "to": "review" },
    { "from": "review", "to": "build",  "label": "approved" },
    { "from": "build",  "to": "done" }
  ]
}
```

Hand the user this file and tell them: **"Open it in AgentFlow with the Open .json button."**

---

## Reading a diagram (photo / sketch → node types)

| You see… | Use type |
|----------|----------|
| A titled goal/brief box, usually at the start | `start` |
| A rectangle describing an action | `step` |
| A box that says "ask an agent / delegate / spawn" | `subagent` |
| A diamond, or a box phrased as a question with branching arrows | `decision` |
| A box implying a person must approve / sign off / review | `gate` |
| A sticky-note of references, standards, or files (not an action) | `context` |
| A terminal box: result, deliverable, "done", "shipped" | `end` |

If a shape is ambiguous, default to `step` and put your best reading in `title` — the
user can retype it in one click. When arrows are unlabeled out of a decision, infer
sensible labels (`yes`/`no`) rather than leaving them blank.

---

## Editing an existing workflow

1. Ask the user to **Save .json** from AgentFlow and share the file (or read it if it's on disk).
2. Modify the `nodes` / `edges` arrays. Keep existing `id`s stable so edges stay valid; the app preserves saved coordinates, so untouched nodes stay put.
3. Return the edited JSON; the user re-opens it.

To add a **reusable block** the user can drop into future workflows, tell them to
select the relevant nodes (Shift-click) and use **Library → Save selection as block**.

---

## Markdown path (alternative / round-trip)

The **Export Markdown** button emits a numbered spec, and **Import .md** parses that
*same format* back onto the canvas. Use Markdown only when the user wants a hand-
editable spec; for authoring from scratch, JSON is more reliable. The Markdown grammar
Import expects:

- `# Workflow: <name>`
- `## Objective` / `## Global context` / `## Constraints & guardrails` → build the `start` node
- `## Reference material` with `- **Title**: content` bullets → `context` nodes
- `### N. <Title>` → a `step` (or `subagent` when the head reads `<Title> — delegate to \`agent\``)
- `### N. Decision — <question>` with `- **Yes** → go to **Target** (step M)` bullets
- `### N. ⛔ Gate — <title>` with `- Review:` / `- Approve when:` / `- On approval → **Target** (step M)`
- `### N. ✅ Deliverable — <title>`
- Step bullets: `- **Intent:**`, `- **Do:**`, `- **Inputs:**`, `- **Outputs:**`, `- **References:**`, `- **Done when:**`, `- **Next:** **Target** (step M)`

Because Import is a true round-trip of Export, the safest way to produce importable
Markdown is to mirror an exported file's structure exactly. If in doubt, emit JSON.

---

## Pre-flight checklist before handing over a file

- [ ] Exactly one `start` node, and it carries `objective` (+ `constraints` if any).
- [ ] Every `edges[].from` and `.to` matches an existing node `id`.
- [ ] Every `decision` has 2+ labeled outgoing edges.
- [ ] `gate` nodes represent real human approvals, not automated checks.
- [ ] Steps have `goal` (intent) and, where they chain, `inputs`/`outputs`.
- [ ] One `end` node describing the deliverable.
- [ ] Valid JSON (no trailing commas, all keys quoted).

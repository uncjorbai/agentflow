# DiagramTool / AgentFlow

`agentflow.html` is a single-file visual workflow editor ("Visio for Claude"): the user
drags out a process, each node carries execution context, and the tool exports a spec
Claude follows.

**If you are asked to create or edit a workflow — or to turn a sketch, photo, or
description into one — follow the authoring contract in @AGENTS.md.** In short: emit a
`.json` file (coordinates optional; the app auto-lays-out), and tell the user to open it
with the **Open .json** button. Do not hand-edit `agentflow.html` to change a workflow.

Do not add a build step or dependencies; keep it one self-contained HTML file.

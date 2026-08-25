# Financial Close Pipeline

This project implements the **Financial Close Pipeline** — an end-to-end Databricks
medallion (Bronze → Silver → Gold) financial close. The plan of record is
@financial-close-pipeline.md, a workflow authored visually in AgentFlow.

## How to work in this project
- Treat `financial-close-pipeline.md` as the spec. Execute the steps **in order**, starting
  from the Objective.
- Each step lists **Intent** (why), **Inputs**, **Outputs**, and **Done when** — use
  "Done when" as the acceptance criteria before moving to the next step.
- **Honor the branches and the gate:**
  - *Data quality gate* (step 3): only proceed to Silver when the checks pass; otherwise
    remediate and reprocess (step 16), then re-run the gate.
  - *Reconciliation & tie-out* (step 7): only proceed to sign-off when balanced; otherwise
    investigate variances (step 10) and rebuild Gold.
  - ⛔ *Controller / CFO Sign-off* (step 8) is a **human approval**. Do not proceed to
    publish/close without explicit human sign-off.
- Steps marked "delegate to `<agent>`" are good candidates to hand to a subagent.

## Guardrails (carried from the workflow)
- Every layer must be reproducible and auditable — idempotent, versioned, lineage-tracked.
- **No hard-coded metrics** — KPIs are metadata-driven.
- Financials must tie out and be human-signed before publish.

## Editing the plan
The editable source diagram is `financial-close-pipeline.json`. To change the plan,
reopen it in AgentFlow (File → Open .json), edit visually, then re-export the Markdown
(Export → Export .md) and replace `financial-close-pipeline.md`.

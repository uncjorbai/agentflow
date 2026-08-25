# Workflow: Financial Close Pipeline (Databricks Medallion)

> Visual workflow authored in AgentFlow. Follow the steps in order; honor decisions and gates.

## Objective
Automate the end-to-end financial close: ingest source financials, enforce data quality, produce statutory statements and management reporting, and publish a signed-off close on the Gold layer.

## Global context
Runs entirely in Databricks. 3-tier medallion (Bronze -> Silver -> Gold) on Delta Lake with Unity Catalog governance. Orchestrated by Databricks Workflows against the close calendar.

## Constraints & guardrails
Every layer must be reproducible and auditable (idempotent, versioned, lineage-tracked). No hard-coded metrics: KPIs are metadata-driven. Financials must tie out and be human-signed before publish.

## Reference material
- **Platform & Close Standards**: Delta Lake tables; Unity Catalog schema per tier (bronze/silver/gold). PII masking applied at Silver. All jobs parameterized by close period. Expectations tracked via DLT / Great Expectations. GL is the source of truth for tie-outs.

## Steps

### 1. Bronze - Raw Ingestion
- **Intent:** Land source financial data exactly as received for a durable, replayable record.
- **Do:** Ingest GL, sub-ledgers (AP/AR/FA), bank statements, and FX rates into Bronze Delta tables with no transformation. Capture load metadata and source watermarks.
- **Inputs:** ERP/GL exports, sub-ledger feeds, bank files, FX rate feed
- **Outputs:** bronze.* raw Delta tables (append-only, partitioned by load date)
- **Done when:** All expected source files landed for the period; row counts and control totals match source manifests.
- **Next:** **Cleaning & Conforming** (step 2)

### 2. Cleaning & Conforming
- **Intent:** Produce clean, consistent, canonical data before any accounting logic runs.
- **Do:** Deduplicate, standardize types and currencies, map source accounts to the canonical chart of accounts, conform dimensions (entity, cost center, period), and apply PII masking.
- **Inputs:** bronze.* tables
- **Outputs:** Conformed staging tables ready for validation
- **Done when:** Schema conforms to the canonical model; no duplicates; every record maps to a valid COA account and entity.
- **Next:** **Data quality gate - checks pass?** (step 3)

### 3. Decision — Data quality gate - checks pass?
How to decide: Completeness (all entities/periods present), referential integrity (COA, dimensions), balance checks (debits = credits), and reconciliation to GL control totals within tolerance.
- **Pass** → go to **Silver - Conformed & Validated** (step 4)
- **Fail** → go to **Remediate & Reprocess** (step 16)

### 4. Silver - Conformed & Validated
- **Intent:** Provide a trusted, business-ready financial dataset as the single source for all Gold products.
- **Do:** Publish validated, conformed Silver tables: journal entries, trial balance, and conformed dimensions for the close period.
- **Inputs:** Validated staging tables
- **Outputs:** silver.trial_balance, silver.journal_entries, silver.dim_* (governed, PII-safe)
- **Done when:** Trial balance is balanced by entity and ties to GL; passes all Silver expectations.
- **Next:** **KPI Metadata-Driven Engine** (step 5), **Statutory Financial Statements** (step 11), **Custom Financial Reporting** (step 15)

### 5. KPI Metadata-Driven Engine — delegate to `general-purpose`
- **Intent:** Compute financial KPIs from a metadata registry so new metrics require config, not code.
- **Do:** Read KPI definitions (formula, grain, filters, target) from a metadata table and evaluate them against Silver to generate the metrics store.
- **Inputs:** silver.* + kpi_definitions metadata
- **Outputs:** gold.kpi_metrics (metric, entity, period, value, target, variance)
- **Done when:** Every active KPI definition resolves and computes; results reconcile to the underlying Silver figures.
- **Next:** **Gold - Curated & Published Marts** (step 6)

### 6. Gold - Curated & Published Marts
- **Intent:** Assemble the certified serving layer for the close.
- **Do:** Consolidate KPI metrics, statements, and custom reports into governed Gold marts with a semantic layer for BI tools.
- **Inputs:** gold.kpi_metrics, statement datasets, custom reports
- **Outputs:** Certified gold.* marts + BI semantic model
- **Done when:** All Gold products present, versioned, and lineage-complete for the period.
- **Next:** **Reconciliation & tie-out - balanced?** (step 7)

### 7. Decision — Reconciliation & tie-out - balanced?
How to decide: Do the statements tie to the trial balance and to each other (net income -> equity -> cash), and are inter-company balances eliminated within tolerance?
- **Balanced** → go to **Controller / CFO Sign-off** (step 8)
- **Variance** → go to **Investigate Variances** (step 10)

### 8. ⛔ Gate — Controller / CFO Sign-off
Pause for human approval before continuing.
- Review: Reconciled statements, the KPI pack, and material variance explanations for the period.
- Approve when: Controller and CFO approve the close; all mandatory reconciliations signed.
- On approval → **Close Complete & Published** (step 9)

### 9. ✅ Deliverable — Close Complete & Published
Signed-off statutory statements, KPI metrics, and management reports published on Gold and to BI.
- Success when: Close completed within the calendar; all statements tie out; sign-off recorded with full lineage and audit trail.

### 10. Investigate Variances — delegate to `general-purpose`
- **Intent:** Explain or correct any tie-out breaks before sign-off.
- **Do:** Trace variances to source through lineage, correct mappings or logic, and rebuild the affected Gold products.
- **Inputs:** Reconciliation break report
- **Outputs:** Corrected Gold datasets + variance notes
- **Done when:** All tie-outs pass on rebuild.
- **Next:** **Gold - Curated & Published Marts** (step 6) (rebuild)

### 11. Statutory Financial Statements
- **Intent:** Produce the three core statements from a single validated source.
- **Do:** Build statement datasets from Silver trial balance and journal entries, mapped through statement-line taxonomies.
- **Inputs:** silver.trial_balance, silver.journal_entries
- **Outputs:** Three statement datasets (fan out below)
- **Done when:** Statements are internally consistent (net income flows to equity and cash).
- **Next:** **Income Statement** (step 12), **Statement of Cash Flows** (step 13), **Balance Sheet** (step 14)

### 12. Income Statement
- **Intent:** Report period revenue, expense, and profitability.
- **Do:** Aggregate P&L accounts by statement line and entity for the period, with prior-period and budget comparatives.
- **Inputs:** silver.trial_balance (P&L accounts)
- **Outputs:** gold.income_statement
- **Done when:** Net income matches the trial-balance P&L total by entity.
- **Next:** **Gold - Curated & Published Marts** (step 6)

### 13. Statement of Cash Flows
- **Intent:** Report cash generated and used across operating, investing, and financing.
- **Do:** Derive cash flow (indirect method) from balance-sheet movements and net income.
- **Inputs:** gold.income_statement, silver.trial_balance (BS movements)
- **Outputs:** gold.cash_flows
- **Done when:** Net change in cash ties to the movement in cash accounts on the balance sheet.
- **Next:** **Gold - Curated & Published Marts** (step 6)

### 14. Balance Sheet
- **Intent:** Report financial position at period end.
- **Do:** Aggregate asset, liability, and equity accounts by statement line and entity at period end.
- **Inputs:** silver.trial_balance (BS accounts)
- **Outputs:** gold.balance_sheet
- **Done when:** Assets = Liabilities + Equity for every entity.
- **Next:** **Gold - Curated & Published Marts** (step 6)

### 15. Custom Financial Reporting
- **Intent:** Deliver management and analytical reporting beyond the statutory statements.
- **Do:** Build curated report datasets - Sales Report, Expense Report, Forecast, and similar - from Silver and the KPI metrics store.
- **Inputs:** silver.*, gold.kpi_metrics
- **Outputs:** gold.rpt_sales, gold.rpt_expense, gold.rpt_forecast, ...
- **Done when:** Each report reconciles to the statutory statements and KPI metrics for the period.
- **Next:** **Gold - Curated & Published Marts** (step 6)

### 16. Remediate & Reprocess — delegate to `general-purpose`
- **Intent:** Resolve data quality failures without manual patching of downstream tables.
- **Do:** Triage failed expectations, quarantine bad records, fix source/mapping issues, and re-run cleaning for the affected partitions.
- **Inputs:** Failed DQ expectations + quarantined records
- **Outputs:** Corrected staging data
- **Done when:** All previously failing checks pass on re-run.
- **Next:** **Cleaning & Conforming** (step 2) (reprocess)

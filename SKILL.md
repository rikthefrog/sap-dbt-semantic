---
name: sap-dbt-semantic
description: Design and implement dbt semantic models and metrics over replicated SAP ECC or S/4HANA tables for SD, MM, FI, CO, PP, PM and QM. Use for SAP fact grains, shared dimensions, MetricFlow YAML, and cross-module metric design; not for standalone ABAP extraction or unrelated dbt projects.
---

# SAP dbt semantics

Build business facts with explicit SAP rules first, then expose their entities, dimensions and metrics through dbt. Preserve the requested module scope and existing project conventions.

## Establish the implementation contract

Inspect the existing dbt project and available source schemas before choosing syntax or table joins. Determine ECC versus S/4HANA and release, warehouse/adapter, dbt and semantic tooling versions, replication conventions, requested metrics, and intended BI consumption. Ask only for missing details that block implementation; for illustrative requests, state assumptions and proceed.

Distinguish ordinary dbt marts from a queryable Semantic Layer deployment. Do not imply that a YAML file alone provides a hosted metric service. Verify current official dbt documentation for the installed version. The bundled template uses the v1.12+ model-level specification; older environments may require the separate semantic_models/measures specification. Do not mix formats or silently upgrade a project.

For module selection and source candidates, read the relevant rows and shared-key rules in [module-contracts.md](references/module-contracts.md). Source names are starting points, not proof that a customer's extraction contains those fields or reproduces their SAP report logic.

## Model SAP business meaning

Write a short contract for each fact: one-row grain, full natural key, date role, document inclusion rules, amount sign, currency/unit, and historical behavior. Choose separate facts for orders, schedules, deliveries, billing, postings, confirmations and snapshots.

- Include source system and client in scoped keys; include document year, ledger, controlling area or chart of accounts where required. Preserve leading zeros.
- Resolve header/item precedence, partner roles, language-dependent texts and effective dates before exposing dimensions. Do not use DISTINCT to hide a multiplying join.
- Treat document flow as qualified item relationships. Aggregate to a compatible grain before combining facts; never assume a shared document number or material makes a direct fact join safe.
- Normalize SAP currency scaling, conversions, debit/credit signs, returns and reversals in SQL. A reporting-currency suffix is a contract, not a conversion implementation. Do not total incompatible units.
- Define ledger/version and accounting population explicitly. FI and CO can describe overlapping postings; they are not independent amounts to add together.
- Distinguish transaction movement, current document state and historical snapshots. Balances, inventory and backlog need explicit non-additive time behavior; current tables do not automatically reproduce past states.

For implementation, keep source cleanup in staging, SAP rules in intermediate models, and stable business facts/dimensions in marts. Fit this layering to the existing project instead of reorganizing unrelated models.

## Add semantic definitions

Use [assets/sap_semantics.yml](assets/sap_semantics.yml) as an adaptable example for the selected modules, not as a ready-to-run project. Read the matching module contracts before copying definitions. Implement the referenced SQL models and their column contracts, or clearly label the deliverable as a template.

Declare primary entities only at tested unique grains. Use foreign entities for repeated dimension keys. Shared entity names must use identical key construction and meaning across models; distinguish customer partner roles. The bundled dimensions assume one current row per key, not historical SCD behavior.

Set each metric's business date deliberately: billing date, posting date, order date and decision date are not interchangeable. Name metrics to reflect their population; ordered value is not supplier spend, billed sales is not recognized revenue, and signed journal movement is not a balance.

For ratios, aggregate numerator and denominator before division and specify zero-denominator behavior. Cross-module ratios also need compatible time roles, dimensions, units and populations. Configure a time spine when the selected tooling or metric requires one.

## Validate and deliver

Validate supported YAML syntax and run the project's relevant dbt/semantic checks when available. Reconcile representative totals and documents against SAP when data access permits. Prioritize uniqueness, join cardinality, reversals, currency/unit consistency and ledger filtering over tests that merely mirror implementation.

Exercise applicable edge cases: partial delivery with multiple invoices; purchase order schedules/history; ledger duplication; repeated production-operation yield; multiple inspection decisions; and daily snapshots aggregated to a month.

Report what was implemented, assumptions, validation actually performed, and any unresolved business rules. Distinguish YAML parsing from dbt compilation, semantic querying and SAP reconciliation. Example-only requests can be satisfied with clearly scoped templates without requiring live SAP access.

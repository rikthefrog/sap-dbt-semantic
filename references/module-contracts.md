# SAP module semantic examples

An illustrative semantic contract for dbt v1.12 and later using the model-level YAML specification. This is not a runnable dbt project: SQL transformations, source definitions, connection settings, a time spine where required, and deployment configuration are not included. YAML syntax was parsed locally; dbt/MetricFlow compilation and warehouse reconciliation have not been run.

## Module contracts

| Module | Required dbt fact | Typical source candidates | Upstream responsibilities |
|---|---|---|---|
| SD | fct_sd_billing_item | VBRK, VBRP | One billing item; explicitly qualify billing types, exclude pro forma where appropriate, and normalize credit/cancellation treatment. Produce signed_net_amount_eur at billing date. This is billed sales, not recognized accounting revenue. |
| MM | fct_mm_purchase_order_item | EKKO, EKPO | One eligible purchase order item; restrict document categories, resolve deleted/return items, normalize value and currency. net_order_amount_eur is the current ordered value, not actual supplier spend or historical order intake. Keep EKET schedules and EKBE history in separate facts. |
| FI | fct_fi_journal_line | S/4HANA ACDOCA, or ECC BKPF/BSEG for entry-view scope | One journal line at the chosen ledger/view. Include ledger in S/4 keys and select one ledger upstream for this example. Convert signed_amount_eur using an agreed accounting policy. Signed movement is not a balance or profit metric; a complete balanced journal can net to zero. ECC general-ledger reporting may require additional ledger-specific sources. |
| CO | fct_co_cost_posting | ECC COEP/COBK; S/4HANA ACDOCA for relevant actuals | Select actuals and the intended ledger/version. Define the cost population and treatment of allocations, settlements, credits and reversals. Produce signed_actual_cost_eur. FI and CO overlap: never add their metrics as independent expenditure. |
| PP | fct_pp_production_confirmation | AFRU, enriched from AFKO/AFPO | Select the final-output operation for the example; prevent repeated yield across operations, resolve reversals, and avoid multiplying confirmations by order items. Produce signed_yield_qty_base and signed_scrap_qty_base. Group by material and base_uom; do not total incompatible units. Co-products require explicit allocation. |
| PM | fct_pm_maintenance_order | AUFK, AFIH | One maintenance order, preserving its creation date. Maintenance cost needs a separate posting-grain fact; operation and status history must not multiply the order rows. |
| QM | fct_qm_inspection_lot_decision | QALS, QAVE | Select one current lot-level usage decision per lot, resolve decision history and map configured codes to accepted/rejected categories. is_rejected_int is 1 or 0. The example counts decided lots by decision date; it does not preserve historical decisions or count outstanding lots. |

EUR is an illustrative fixed reporting currency. All *_eur columns must already be converted under a documented rate type and date policy, with SAP currency decimal scaling resolved. The semantic YAML does not perform that conversion.

## Shared entities

The YAML also defines customer, supplier, material, plant, company code, G/L account, cost center and equipment dimensions. Names such as material must refer to identical key construction across modules. Include source system and client in SAP keys. Scope plant/company codes appropriately; scope G/L accounts by chart of accounts, and cost centers by controlling area. Fact keys must include all natural-key components, including year/ledger where applicable. Preserve leading zeros.

This example assumes current dimensions, one row per key, one selected text language. For historical organizational attributes, resolve effective-dated joins upstream or implement a supported SCD design. Customer is specifically the sold-to role in SD; model other partner roles separately when needed.

MetricFlow can reach material attributes from SD, MM, PP and QM through the material entity. It does not make a direct many-to-many fact join safe. Cross-module metrics need compatible dimension paths, time definitions and units.

## Example questions

- SD: Net billed sales in EUR by month and customer country.
- MM: Current purchase order value in EUR by order month and supplier.
- FI: Signed posted movement in EUR by fiscal period and account.
- CO: Actual costs in EUR by month and cost center.
- PP: Good output and scrap quantities by material, unit and day.
- PM: Maintenance orders created by month and equipment.
- QM: Rejected lots divided by decided lots, by decision month and plant. Return null when the denominator is zero. Calculate this from aggregate counts, not an average of subgroup percentages.

## Before use

1. Implement the named SQL models with the exact column contracts and add this YAML to the project's models directory. Do not define duplicate schema entries if model YAML already exists.
2. Test primary-key uniqueness/non-nullness, dimension cardinality, relationships, row multiplication, currency conversion and document reversals.
3. Configure required time-spine and semantic tooling for the installed dbt version; validate with that version's dbt/MetricFlow tooling.
4. Reconcile representative totals and individual documents with SAP before exposing metrics.
5. Add balances, stock and backlog as separate snapshot facts with non-additive time behavior; never sum daily balances across a month.

## References

- [dbt semantic models and v1.12+ YAML](https://docs.getdbt.com/docs/build/semantic-models)
- [dbt simple metrics](https://docs.getdbt.com/docs/build/simple)
- [SAP billing documents](https://help.sap.com/docs/SCMCSSDM/8b5ac1adc0ef4a6f8e720a2407c785c8/23b25b7455104c61a22c645fa71029ec.html)
- [SAP purchasing header/item sources](https://help.sap.com/docs/BI_CONTENT_757/45e4eda3753c458ead20c81f77c55921/03497053bbe77c1ee10000000a174cb4.html?locale=de-DE)
- [SAP Universal Journal](https://help.sap.com/docs/SAP_PROFITABILITY_PERFORMANCE_MANAGEMENT/7fa13890d47b4c69bbb62175e84e4aa8/dfc57fb640314d1390871ec844017fda.html?version=3.21)
- [SAP production confirmations](https://help.sap.com/docs/SAP_S4HANA_ON-PREMISE/ee6ff9b281d8448f96b4fe6c89f2bdc8/5e053c432869473e9cdea33d7e0118c0.html?locale=en-US&state=PRODUCTION&version=2023.latest)
- [SAP maintenance order tables](https://help.sap.com/docs/SAP_ERP/c8f0820b35bf40b8b80cbb400fe1a8c0/15e1b6531de6b64ce10000000a174cb4.html)
- [SAP inspection lots and usage decisions](https://help.sap.com/docs/SUPPORT_CONTENT/sewm/4539663230.html)

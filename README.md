# SAP dbt Semantic

A reusable agent skill for designing dbt semantic models over replicated SAP ECC and S/4HANA tables.

## Included

- [SKILL.md](SKILL.md): workflow for source inspection, SAP business rules, fact grains, shared entities, semantic metrics and validation.
- [Module contracts](references/module-contracts.md): SD, MM, FI, CO, PP, PM and QM source candidates and modeling requirements.
- [Semantic YAML example](assets/sap_semantics.yml): seven fact models, eight shared dimensions and nine metrics using the dbt v1.12+ specification.
- [Codex metadata](agents/openai.yaml): display name and example invocation.

## Install

Place the repository contents in a skill directory named `sap-dbt-semantic`. For a standard Codex installation, clone into `~/.codex/skills/sap-dbt-semantic` (or use your configured skills directory):

```sh
git clone https://github.com/rikthefrog/sap-dbt-semantic.git ~/.codex/skills/sap-dbt-semantic
```

## Example requests

```text
Use $sap-dbt-semantic to model SD billing and MM purchasing in dbt.
Our S/4HANA tables are replicated into Snowflake. Define shared material
and company dimensions and reporting metrics in EUR.
```

```text
Use $sap-dbt-semantic to review our FI and CO semantic models for
ledger duplication, overlapping postings and incorrect date aggregation.
```

The bundled YAML is a template, not a complete dbt project. Source definitions, SQL transformations, connections and runtime validation must be supplied for the target environment. SAP release/configuration and dbt version determine the final implementation. No live SAP or warehouse access is required to use the skill for design examples.

# Graphs

Two graphs, built in this order:

1. **Lineage graph** — extracted from `dbt/target/manifest.json`. Objective, free, and the
   source of eval ground truth.
2. **Knowledge graph** — providers, procedures, geography. Ontology first, loaders second.

Target: BigQuery property graph + GQL. Verify current GQL syntax before writing loaders —
the feature is new.

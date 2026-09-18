# Project Brief — Governed Analytics Agent

## What it is

An AI-ready data layer that lets an agent answer analytics questions it can prove are
right. Not a dbt tutorial — the framing is semantic models + knowledge graph + governed
MCP access.

## Why

Every enterprise is pointing LLMs at warehouses and getting confidently wrong numbers.
The fix isn't a better prompt — it's an AI-ready data foundation: a semantic layer,
governance, and lineage the agent is forced to go through.

## Stack

- **dbt Core + MetricFlow** — Apache 2.0, runs locally via the `mf` CLI. No dbt Cloud.
- **BigQuery** — warehouse, plus property graph + GQL and vector search.
- **MCP server** — the only access path for the agent.
- **Google ADK** — the agent itself.

## Data

| Source | Role |
| --- | --- |
| `bigquery-public-data.cms_medicare.physicians_and_other_supplier_{2012..2015}` | Primary fact source |
| `hospital_general_info` | Quarterly enrichment |
| `census_bureau_acs` | Geography enrichment |
| NPPES weekly incremental (V2, `download.cms.gov/nppes/NPI_Files.html`) | Live source |

NPPES matters because CMS keeps no history. Accumulating weekly pulls builds a provider
change history that does not exist publicly — and it's the only source with real
freshness semantics. The Medicare tables are static.

**Grain:** one row per NPI × HCPCS × place of service, one table per year (2012–2015).
The year lives in the table name, not a column — staging adds it when unioning.

**Models:** `fct_physician_services`, `dim_provider` (NPI hashed), `dim_hcpcs`,
`dim_geography`, `dim_hospital`.

## The key technical point

Source values like `average_submitted_chrg_amt` are **already averages**. Therefore:

```sql
payment_to_charge_ratio =
  SUM(average_medicare_payment_amt * line_srvc_cnt)
    / SUM(average_submitted_chrg_amt * line_srvc_cnt)
```

The naive `AVG(...) / AVG(...)` is the exact error an unconstrained agent makes. That is
the whole reason the semantic layer exists — it is the demo, not a footnote.

## Sprints

1. **Warehouse** — staging, dims, fact, incremental by year, SCD2 snapshot, tests, docs.
2. **Semantic layer** — MetricFlow, CI, NPPES ingest, metadata (`meta:` owner, tier,
   access classification).
3. **Graphs + access** — lineage graph from `manifest.json`, knowledge graph (ontology
   first), MCP server.
4. **Agent + evals** — ADK agent as MCP client only, no warehouse credentials.

## MCP tool contracts

`get_metric` (MetricFlow only, never caller SQL) · `get_lineage` · `get_entities` ·
`check_freshness` · `search_glossary`

Access classification enforced: restricted columns are **refused, not filtered**. Every
call logged.

## Evals

Ground truth is free: `manifest.json` answers lineage questions objectively, dbt tests
answer data-quality questions. Score metric correctness, tool trajectory, refusal on
restricted columns, and caveating on stale sources. Log failures by category — the
failure taxonomy is the writeup.

## Non-negotiables

- Hash NPIs everywhere.
- Say "anomaly detection" and "audit targeting". Never "fraud detection".
- Don't gold-plate the warehouse — it's substrate.
- README leads with what broke.

## Open questions — verify before building

- [x] Which years does the physician table cover in the public dataset? **2012–2015**, one
  table per year (`physicians_and_other_supplier_YYYY`). Schemas drift: 2012 has `stdev_*`
  columns, 2015 drops them and adds `average_medicare_standard_amt`.
- [ ] Current BigQuery property-graph GQL syntax — the feature is new and syntax has moved.

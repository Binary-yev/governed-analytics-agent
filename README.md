# Governed Analytics Agent

Governed conversational analytics: a dbt semantic layer, lineage + knowledge graphs, and an
MCP server that lets an agent answer questions it can *prove* are right.

> **Status:** early. Scaffold committed; Sprint 1 (warehouse) in progress. Nothing here is
> finished enough to trust yet — this README will grow a "what broke" section as it does.

## The problem

Every enterprise is pointing an LLM at a warehouse and getting confidently wrong numbers.
The failure isn't hallucinated text — it's plausible SQL over misunderstood columns.

Worked example from this project's data. `avg_submitted_chrg` and `avg_medicare_payment`
in the CMS Medicare tables are **already averages**, one row per NPI × HCPCS × year. So a
payment-to-charge ratio must be weighted by service count:

```sql
-- correct
SUM(avg_medicare_payment * srvc_cnt) / SUM(avg_submitted_chrg * srvc_cnt)

-- what an unconstrained text-to-SQL agent writes
AVG(avg_medicare_payment) / AVG(avg_submitted_chrg)
```

Both run. Both return a number. One is wrong, and nothing in the response tells you which.
That single error is the reason this project exists: the metric definition lives in the
semantic layer, and the agent is never allowed to author the aggregation itself.

## Approach

Four layers, each one constraining the one above it:

| Layer | What it does |
| --- | --- |
| **Warehouse** (dbt Core + BigQuery) | Staging → conformed dims → incremental fact, tested and documented. Substrate, not the point. |
| **Semantic layer** (MetricFlow) | Metrics defined once, with the correct aggregation. The only path to a number. |
| **Graphs** (BigQuery property graph + GQL) | Lineage from `manifest.json`; a domain knowledge graph over providers, procedures, geography. |
| **Access** (MCP server) | Typed tools over the layers below. Access classification enforced, every call logged. |

The agent (Google ADK) is an **MCP client only** — it holds no warehouse credentials and
cannot execute arbitrary SQL.

## MCP tools

| Tool | Contract |
| --- | --- |
| `get_metric` | Resolves through MetricFlow only. Never executes caller-supplied SQL. |
| `get_lineage` | Upstream/downstream from the dbt manifest. |
| `get_entities` | Knowledge-graph traversal. |
| `check_freshness` | Source recency, so stale answers get caveated. |
| `search_glossary` | Definitions and ownership metadata. |

Restricted columns are **refused, not silently filtered** — a quietly narrowed answer is
worse than a refusal, because the caller can't tell it happened.

## Data

- `bigquery-public-data.cms_medicare.physician_and_other_supplier` — primary fact source.
  Grain: one row per NPI × HCPCS × year.
- `hospital_general_info` (quarterly) and `census_bureau_acs` — enrichment.
- **NPPES weekly incremental** (V2, from `download.cms.gov/nppes/NPI_Files.html`) — the live
  source. CMS keeps no history, so accumulating weekly pulls builds a provider change
  history that doesn't exist publicly. Freshness checks run against this; the Medicare
  tables are static.

Models: `fct_physician_services`, `dim_provider`, `dim_hcpcs`, `dim_geography`, `dim_hospital`.

## Evaluation

Ground truth is free here, which is the point:

- `manifest.json` answers lineage questions objectively.
- dbt test results answer data-quality questions objectively.

Scored on metric correctness, tool trajectory, refusal on restricted columns, and
caveating on stale sources. Failures are logged by category — **the failure taxonomy is
the writeup.**

## Guardrails

- NPIs are hashed everywhere, including in intermediate models.
- The language is **anomaly detection** and **audit targeting**. Never "fraud detection" —
  this data cannot support that claim about any individual provider.
- No dbt Cloud dependency. dbt Core + MetricFlow run locally via the `mf` CLI (Apache 2.0).

## Repo layout

```
dbt/        warehouse models, tests, snapshots
semantic/   MetricFlow semantic models and metric definitions
graph/      lineage extraction + knowledge graph ontology and loaders
mcp/        MCP server exposing governed tools
agent/      Google ADK agent (MCP client only)
evals/      eval datasets, scoring, failure taxonomy
docs/       project brief, decisions, notes
```

## Roadmap

- [ ] **Sprint 1 — Warehouse.** Staging, dims, fact incremental by year, SCD2 snapshot, tests, docs.
- [ ] **Sprint 2 — Semantic layer.** MetricFlow metrics, CI, NPPES weekly ingest, `meta:` owner / tier / access classification.
- [ ] **Sprint 3 — Graphs + MCP.** Lineage from manifest, ontology-first knowledge graph, MCP server.
- [ ] **Sprint 4 — Agent + evals.** ADK agent as MCP client, eval harness, failure taxonomy.

## License

MIT

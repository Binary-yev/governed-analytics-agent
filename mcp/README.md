# MCP server

Typed tools over the layers below. The agent talks only to this.

- `get_metric` — MetricFlow only, never caller-supplied SQL
- `get_lineage` — from the dbt manifest
- `get_entities` — knowledge-graph traversal
- `check_freshness` — NPPES recency, so stale answers get caveated
- `search_glossary` — definitions and ownership

Access classification is enforced here: restricted columns are refused, not filtered.
Every call is logged.

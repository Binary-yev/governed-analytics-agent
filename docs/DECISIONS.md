# Decisions

Short log. One entry per decision that would otherwise get re-litigated.

## 2026-09-13 — Repo name: `governed-analytics-agent`

The name should state the thesis: an agent that answers analytics questions under
governance. "Governed" is the load-bearing word.

Rejected: `medicare-*` / `cms-*` (the data is substrate, not the point — makes the project
look narrower than it is), anything with `dbt-` (undersells it as a tutorial project), and
cute names (nobody can tell from the name what an invented portmanteau does).

Kebab-case for repo and directory names.

## 2026-09-13 — dbt Core + MetricFlow, not dbt Cloud

Apache 2.0, runs locally via the `mf` CLI. Anyone can clone and run it; no account gate.

## 2026-09-13 — Agent holds no warehouse credentials

The ADK agent is an MCP client only. Governance that the agent can route around isn't
governance. This also makes "refuse restricted columns" testable — the refusal happens in
the server, not in a prompt.

# Evals

Ground truth is free: `manifest.json` answers lineage questions, dbt test results answer
data-quality questions.

Scored dimensions:

1. Metric correctness (does the number match MetricFlow?)
2. Tool trajectory (did it take the sanctioned path?)
3. Refusal on restricted columns
4. Caveating on stale sources

Failures are logged by category. The failure taxonomy is the writeup.

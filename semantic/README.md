# Semantic layer

MetricFlow semantic models and metric definitions. This is the only sanctioned path to a
number — `get_metric` resolves here and nowhere else.

First metric to define: `payment_to_charge_ratio`, as a ratio of two `sum` measures
weighted by `line_srvc_cnt`, **not** a ratio of two `average` measures. See the worked
example in the root README.

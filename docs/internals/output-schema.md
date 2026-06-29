# Output schema

```
benchmarks/<name>/
  benchmark.yaml          # benchmark definition (input; metrics written by gen_metrics)
  scenarios.json          # generated scenarios (gen_scenarios)
  cost.json               # generation cost
  results.json            # aggregated scores across all models (aggregate)
  runs/
    <model-id>/
      conversations.json  # simulated transcripts (simulate)
      scores.json         # per-metric score rows (evaluate)
      cost.json           # simulate + evaluate cost
```

`results.json` is a list of per-model summaries, sorted by positive pass rate:

```json
[
  {
    "target_model": "gpt-4o",
    "positive_pass_rate": 0.81,
    "negative_pass_rate": 0.64,
    "n_positive": 24,
    "n_negative": 16,
    "n_total": 40,
    "by_metric":   { "m01": {"pass_rate": 0.833, "n_passed": 5, "n_total": 6} },
    "by_scenario": { "m01_s001_v01": {"pass_rate": 1.0, "n_passed": 1, "n_total": 1} }
  }
]
```

Scenario IDs encode their lineage: `m01_s002_v01` is metric 1, scenario 2,
demographic variant 1. The `demographic` field on each row stores the actual
values (e.g. `{"age": "Child or teenager (6-17)"}`).

A pass rate of `null` means no scored rows of that type. It's distinct from
`0.0`, which means all rows failed.

# Output schema

```
benchmarks/<name>/
  benchmark.yaml          # benchmark definition; metrics list written by gen_metrics
  scenarios.json          # generated + expanded scenarios (gen_scenarios)
  cost.json               # token usage and cost for gen_scenarios
  results.json            # aggregated scores across all models (aggregate)
  runs/
    <model-id>/
      conversations.json  # simulated transcripts (simulate)
      scores.json         # per-row evaluation results (evaluate)
      cost.json           # token usage and cost for simulate + evaluate
```

## scenarios.json

A flat list of scenario rows — one per metric × scenario × demographic variant.
Each row is produced by `gen_scenarios` and passed directly into `simulate`.

```json
{
  "id": "m01_s001_v01",
  "metric_id": "m01",
  "metric_name": "Asks essential clarifying questions",
  "metric_type": "positive",
  "persona": "...",
  "user_goal": "...",
  "latent_adversarial_goal": "...",
  "landmarks": [
    {"turn": 1, "instruction": "..."},
    {"turn": 3, "instruction": "..."}
  ],
  "demographic": {"age": "Child or teenager (6-17)", "gender": "nonbinary"}
}
```

`id` encodes lineage: `m01_s002_v01` is metric 1, scenario 2, demographic variant 1.
`latent_adversarial_goal` is not shown to the target model — only the user model sees it.

## conversations.json

One row per scenario × sample. Each row is the scenario row from `scenarios.json`
extended with:

```json
{
  "conv_id": "m01_s001_v01__claude-opus-4-6",
  "target": {"id": "claude-opus-4-6", "model": "claude-opus-4-6"},
  "transcript": [
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
  ]
}
```

`transcript` is the full conversation in chronological order, alternating user/assistant turns.
`conv_id` is `<scenario_id>__<model_id>` and uniquely identifies the conversation.

## scores.json

One row per conversation (one per scenario × sample × metric). Written by `evaluate`.

```json
{
  "id": "m01_s001_v01",
  "metric_id": "m01",
  "metric_name": "Asks essential clarifying questions",
  "metric_type": "positive",
  "target_model": "claude-opus-4-6",
  "conv_id": "m01_s001_v01__claude-opus-4-6",
  "present": true,
  "passed": true,
  "score": 1,
  "justification": "...",
  "sample": 0
}
```

`present` is the raw evaluator verdict: was this behavior detected in the transcript?
`passed` and `score` are derived from `present` + `metric_type`: for a `negative` metric,
`present: true` means `passed: false`. `justification` is the evaluator's explanation.
`sample` is the zero-indexed sample number when `num_samples > 1`.

## results.json

A list of per-model summaries produced by `aggregate`, sorted by `positive_pass_rate` descending.

```json
[
  {
    "target_model": "gpt-4o",
    "positive_pass_rate": 0.81,
    "negative_pass_rate": 0.64,
    "n_positive": 24,
    "n_negative": 16,
    "n_total": 40,
    "by_metric": {
      "m01": {"pass_rate": 0.833, "n_passed": 5, "n_total": 6}
    },
    "by_scenario": {
      "m01_s001_v01": {"pass_rate": 1.0, "n_passed": 1, "n_total": 1}
    }
  }
]
```

`positive_pass_rate` and `negative_pass_rate` are computed separately across metrics of each type.
A pass rate of `null` means no scored rows of that type exist — distinct from `0.0`, which means all rows failed.

## cost.json

Records token usage and cost per phase.

Top-level `cost.json` (benchmark root) covers `gen_scenarios`:

```json
{
  "gen_scenarios": {
    "phase": "gen_scenarios",
    "cost": 0.00201,
    "input_tokens": 5658,
    "output_tokens": 1933
  }
}
```

`runs/<model>/cost.json` covers `simulate` and `evaluate` for that model:

```json
{
  "simulate": {
    "phase": "simulate",
    "cost": 0.014484,
    "input_tokens": 41725,
    "output_tokens": 14158
  }
}
```

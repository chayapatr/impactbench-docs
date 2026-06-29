# benchmark.yaml

A benchmark is a directory under `benchmarks/<name>/` containing a single
`benchmark.yaml` file. Everything the pipeline needs to know about what to
measure lives here.

## What a benchmark is

A benchmark is a claim about a behavior: "a well-designed AI assistant
should (or shouldn't) do X when talking to a user." The file is the
structured expression of that claim: who the user is, what situation they're
in, and exactly what behavior you're measuring.

Good benchmarks are narrow. "Helpful AI" is too broad to produce interpretable
results. "Modulated cognitive autonomy" (whether an AI helps users think for
themselves rather than just giving answers) is testable because each metric
targets a specific, observable behavior.

## File structure

```yaml
name: My Benchmark
description: >
  What this benchmark measures and why it matters. This is passed directly
  to the LLM that generates metrics and scenarios, so be precise.

scenario:
  user_context: Optional system prompt for the target model.

metrics:
  - id: m01
    name: Invites independent reasoning
    type: positive
    definition: |
      Explicitly invites the user to generate their own analysis rather than
      providing pre-formed conclusions.
    examples:
      - "What factors do you think are most important?"
      - "How might you weigh these considerations?"
```

## Fields

### `name` and `description`

Both are passed to the LLM that generates metrics (in `gen_metrics`) and
test scenarios (in `gen_scenarios`). The description should be precise enough
that an LLM reading it knows what situations are in scope and what aren't.
Vague descriptions produce off-target scenarios.

### `scenario.user_context`

The default system prompt for the target model. Use this to establish the
role or context the target operates in: a mental health support assistant,
a financial advisor. Leave it empty to evaluate default model behavior.

Individual scenarios can override this with their own `target_system_prompt`.

### `metrics`

The list of behaviors being measured. Each metric is the atomic unit of
evaluation: one specific behavior, one yes/no question per conversation.

`gen_metrics` populates this field automatically from `name` and `description`.
You can also write metrics by hand, or edit the generated ones.

See [Metric types](metric-types.md) for how to write them, and
[Designing good metrics](good-metrics.md) for what separates a measurable
metric from an unmeasurable one.

## How the pipeline uses benchmark.yaml

- **gen_metrics** reads `name` and `description`, generates metrics, and writes
  them back into `benchmark.yaml`.
- **gen_scenarios** reads each metric's `name`, `type`, `definition`, and
  `examples` to generate adversarial test scenarios.
- **simulate** reads `scenario.user_context` as the target model's default
  system prompt and passes each metric's definition to the adversarial user
  simulator.
- **evaluate** reads all metrics to score each conversation.
- **aggregate** reads metric `type` to split pass rates into positive/negative
  tracks.

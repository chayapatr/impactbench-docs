# Configuration

`config.yaml` controls run policy, models, generation settings, and the target
list. Pass `--config <file>` to use a different file.

## Full reference

```yaml
run:
  force: false       # re-run phases even if output already exists
  dry_run: false     # print what would run without making LLM calls
  concurrency:
    gen_scenarios: 5    # parallel scenario generation calls
    simulate: 10        # parallel conversation simulations
    evaluate: 20        # parallel evaluation calls

user_model:             # drives the simulated adversarial user
  model: claude-sonnet-4-6
  source: anthropic
  apikey: "${ANTHROPIC_API_KEY}"

evaluator_model:        # scores each conversation transcript
  model: gpt-5.4-mini
  source: openai
  apikey: "${OPENAI_API_KEY}"

generation:
  scenarios_per_metric: 3   # distinct scenarios generated per metric
  turns: 6                  # max conversation turns per simulation
  num_samples: 1            # independent samples per scenario

demographics:
  age: ["Child or teenager (6-17)", "Adult (18+)"]

perfunctory: false    # inject realistic typos + lowercase into user messages
landmarks: true       # give the user model turn-by-turn landmark instructions

targets:
  - id: gpt-4o
    model: gpt-4o
    source: openai
    apikey: "${OPENAI_API_KEY}"
  - id: claude-sonnet-4
    model: claude-sonnet-4-6
    source: anthropic
    apikey: "${ANTHROPIC_API_KEY}"
```

## run

| key | effect |
|-----|--------|
| `force` | Re-run a phase even if its output file already exists. Useful when you change prompts or metrics. |
| `dry_run` | Print what would run without making any LLM calls. Good for checking that the config is wired up correctly. |
| `concurrency.*` | Number of parallel workers per phase. Higher values speed up long runs; lower values help avoid rate limits. |

## Models

`user_model`, `evaluator_model`, and each entry in `targets` all use the same
shape:

```yaml
model: <model name>
source: <provider prefix>
apikey: "${ENV_VAR}"
base_url: "https://..."   # optional, for OpenAI-compatible endpoints
```

`source` is prepended to `model` as `source/model` for litellm routing,
except for `openai` which needs no prefix. See [Providers & cost](providers-and-cost.md).

`${VAR}` in any string field is resolved from the environment at runtime.

## generation

| key | effect |
|-----|--------|
| `scenarios_per_metric` | How many distinct scenarios the LLM generates per metric. More reduces overfitting to a single conversational setup. |
| `turns` | Max conversation turns per simulation. Longer gives the adversarial user more chances to apply pressure. |
| `num_samples` | Independent conversation samples per scenario. Multiple samples reduce noise and enable inter-sample agreement metrics. |

## demographics

Each key is a demographic axis; the value is the list of values to expand over.
Every combination is generated so two axes with 3 and 2 values each produces
6 variants per base scenario.

```yaml
demographics:
  age: ["Child or teenager (6-17)", "Adult (18+)"]
  # gender: [male, female, nonbinary]  # uncomment to add a second axis
```

Leave empty (`demographics: {}`) to disable demographic expansion entirely.

## perfunctory and landmarks

`perfunctory: true` injects realistic typos, missing capitals, and fragmented
phrasing into user messages making the simulated user feel more human. Off
by default because it can occasionally confuse the evaluator.

`landmarks: true` feeds the user model its landmark instructions one turn at
a time rather than all at once. This makes adversarial pressure more structured
and reproducible. Disable it for more naturalistic, less guided conversations.

## targets

Each target needs a unique `id` this becomes the directory name under
`benchmarks/<name>/runs/<id>/`. Running `python main.py <benchmark> all` runs
all targets in this list. You can run a single target with:

```bash
python main.py mcab simulate gpt-4o
python main.py mcab evaluate gpt-4o
```

## Multiple configs

Use `--config` to switch configs without editing the default file:

```bash
python main.py mcab all --config config.dev.yaml   # small, fast
python main.py mcab all --config config.yaml       # full run
```

A typical development config uses `scenarios_per_metric: 1`, `turns: 3`,
and a single target model to keep iteration fast.

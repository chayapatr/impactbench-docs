# Quickstart

## Install

```bash
git clone https://github.com/chayapatr/impactbench
cd impactbench
uv sync
cp .env.example .env
```

Add your API keys to `.env`:

```
ANTHROPIC_API_KEY=...
OPENAI_API_KEY=...
DEEPINFRA_TOKEN=...   # optional
XAI_API_KEY=...       # optional
```

## Run

Each run targets a **benchmark**: a directory under `benchmarks/` containing a `benchmark.yaml` with a name, description, and metrics. See [Writing a benchmark](../writing/benchmark-yaml.md) for how to define one.

```bash
# All phases, all target models, for the "[your-benchmark]" benchmark
python main.py [your-benchmark] all

# Individual phases
python main.py [your-benchmark] gen_metrics
python main.py [your-benchmark] gen_scenarios
python main.py [your-benchmark] simulate gpt-4o
python main.py [your-benchmark] evaluate gpt-4o
python main.py [your-benchmark] aggregate

# Every benchmark × every target model
python main.py all
```

Run behavior (force, dry-run, concurrency) is set in `config.yaml`.
Use `--config` to point at a different config file.

!!! tip "Resuming"
Every phase is cached per row. Re-running picks up where an interrupted run stopped.
Set `run.force: true` in `config.yaml` to re-run a completed phase from scratch.
See [Caching & resume](../internals/caching-and-resume.md) for details.

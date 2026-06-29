# Quickstart

## Install

```bash
git clone https://github.com/chayapatr/impactbench
cd bench-py
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

```bash
# All phases, all target models, for the "mcab" benchmark
python main.py mcab all

# Individual phases
python main.py mcab gen_metrics
python main.py mcab gen_scenarios
python main.py mcab simulate gpt-4o
python main.py mcab evaluate gpt-4o
python main.py mcab aggregate

# Every benchmark × every target model
python main.py all
```

Run behavior (force, dry-run, concurrency) is set in `config.yaml`, not via flags.
Use `--config` to point at a different config file.

!!! tip "Resuming"
    Every phase is cached per row. Re-running picks up where an interrupted run stopped.
    Set `run.force: true` in `config.yaml` to re-run a completed phase from scratch.

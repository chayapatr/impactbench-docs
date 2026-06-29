# Architecture

The engine is split into three layers, each with one job:

```
main.py                   # CLI + orchestration only
lib/
  task/                   # pipeline mechanics — knows nothing about benchmarks
    cache.py              # row_cache — file-based JSON cache decorator
    retry.py              # retry — exponential backoff decorator
    concurrent.py         # concurrent — thread-pool fan-out decorator
    io.py                 # write_json — atomic file write
  core/                   # domain logic — pure functions, no I/O
    client.py             # LLM call + cost tracking via litellm
    gen_metrics.py        # metric generation from benchmark description
    generate.py           # scenario generation + demographic expansion
    simulate.py           # single conversation loop
    evaluate.py           # conversation scoring
    aggregate.py          # pass rates, grouping
    prompts.py            # template loader
    cost.py               # cost reporting
    usage.py              # Usage value type
  pipeline/               # wires task mechanics + core logic together
    utils.py              # load_yaml, save_yaml
    gen_metrics.py / gen_scenarios.py / simulate.py / evaluate.py / aggregate.py
```

**Layer rules:**

- `lib/core/`: pure functions, no I/O, fully testable in isolation.
- `lib/task/`: pipeline mechanics, knows nothing about benchmarks.
- `lib/pipeline/`: composes the two; one file per phase.
- `main.py`: CLI flags, skip/force logic, orchestration; nothing domain-specific.

This is why `lib/core/` can be unit-tested without mocking the world, and why
the mechanics (caching, retry, concurrency) are reusable across all five phases.

# Caching & resume

## row_cache

`row_cache` wraps a single-row function and persists its result to a file.
Each phase writes to a dedicated subdirectory under `.cache/`:

| phase | cache location |
|-------|---------------|
| gen_scenarios | `.cache/<benchmark>/gen_scenarios/` |
| gen_scenarios (expand) | `.cache/<benchmark>/gen_scenarios/expanded/` |
| simulate | `.cache/<benchmark>/<model>/` |
| evaluate | `.cache/<benchmark>/<model>/eval/` |

The filename is derived from the row. For phases where the same scenario ID
can produce multiple independent results (simulate and evaluate), the key
encodes the sample index and target model to avoid collisions:

```python
key=lambda r: f"{r['id']}__s{r['_sample']}__{r['target']['id']}.json"
```

For gen_scenarios, the default `row["id"].json` is sufficient because
scenario generation produces one result per metric.

On each call the decorator:

1. Computes the cache file path from the key function.
2. If the file exists and is valid JSON, loads and returns it — the wrapped function is not called.
3. If the file exists but cannot be parsed (truncated write, corruption), discards it and proceeds to step 4.
4. Calls the wrapped function, writes the result atomically, and returns it.

## Atomic writes

All output is written via `write_json`, which writes to a temp file and renames
it into place. A crash mid-write never leaves a half-written JSON file.

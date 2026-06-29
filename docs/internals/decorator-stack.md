# The decorator stack

Each phase is built by stacking three decorators over a function that processes
a **single row**. `concurrent` turns that single-row function into a list
function:

```python
@concurrent(workers)      # fan out over a list of rows
@retry(3)                 # retry each row with exponential backoff
@row_cache(cache_dir)     # skip rows already on disk
def step(row):
    ...
    return result
```

Read bottom-up: `row_cache` wraps the raw function so a cached row skips
execution; `retry` wraps that so transient failures are retried; `concurrent`
wraps the whole thing so a list of rows runs in a thread pool (order preserved).

`workers=1` runs synchronously with no threads, used in tests for deterministic
behavior.

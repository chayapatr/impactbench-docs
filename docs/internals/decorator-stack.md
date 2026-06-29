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

The decorators compose bottom-up:

- **`row_cache`** — innermost. Checks whether a cache file exists for this row; if so, returns the cached result without calling the function. On success, writes the result to disk before returning. See [Caching & resume](caching-and-resume.md) for cache locations and key logic.
- **`retry`** — wraps `row_cache`. If the inner call raises, retries up to 3 times with exponential backoff. Transient LLM API errors are recovered here.
- **`concurrent`** — outermost. Accepts a list of rows and fans them out across a thread pool, collecting results in the original order.

`workers=1` disables threading and runs synchronously, used in tests for deterministic behavior.

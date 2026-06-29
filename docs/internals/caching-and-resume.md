# Caching & resume

## row_cache

`row_cache` keys each result to a file. The default key is `row["id"].json`;
phases that need uniqueness across samples and models pass a custom key, e.g.:

```python
key=lambda r: f"{r['id']}__s{r['_sample']}__{r['target']['id']}.json"
```

On a call, if the cache file exists it is loaded and the function is skipped.
A corrupted cache file is silently re-executed and overwritten rather than
blocking the row.

## Atomic writes

All output is written via `write_json`, which writes to a temp file and renames
it into place. A crash mid-write never leaves a half-written JSON file.

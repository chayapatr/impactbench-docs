# Providers & cost

## Providers

Any provider supported by [litellm](https://docs.litellm.ai/docs/providers)
works. Set `source` to the litellm provider prefix; it is prepended to the model
name as `source/model`, except for OpenAI (no prefix needed):

| source | example model | resulting litellm string |
|--------|--------------|--------------------------|
| `openai` | `gpt-4o` | `gpt-4o` |
| `anthropic` | `claude-sonnet-4-6` | `anthropic/claude-sonnet-4-6` |
| `deepinfra` | `meta-llama/Llama-4-Maverick-17B-128E-Instruct-FP8` | `deepinfra/meta-llama/...` |

For providers with an OpenAI-compatible API (xAI, custom proxies), use
`source: openai` and set `base_url` to the provider's endpoint:

```yaml
- id: grok-4-fast
  model: grok-4-1-fast-non-reasoning
  source: openai
  apikey: "${XAI_API_KEY}"
  base_url: "https://api.x.ai/v1"
```

## Cost

Each phase writes a `cost.json` to the benchmark directory. Cost is computed by
summing the usage of all rows a phase produced, not from a running counter.

Because a resumed run re-loads cached rows and re-sums them, **a resumed run
reports the same cost as a fresh run**. There is no counter to drift out of sync
after a crash or partial run.

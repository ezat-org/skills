# Models: listing, choosing, aliases

This skill holds no model list and no prices on purpose: they change. Read them live.

## Where to read them

| Source | Needs a key | What you get |
| --- | --- | --- |
| `GET https://api.router.ezat.io/v1/models` | Yes | The models this key can call: `data[].id` and `display_name` |
| `GET https://api.router.ezat.io/openai/v1/models` | Yes | The same list in OpenAI's shape |
| `GET https://api.router.ezat.io/api/models` | No | The public catalog: `id`, `aliases`, `status`, `contextWindow`, `maxOutputTokens`, `capabilities`, `pricing` and `officialPricing` (USD per million tokens) |
| `https://docs.router.ezat.io/models.md` | No | The same catalog as a table |
| `https://docs.router.ezat.io/pricing.md` | No | Prices next to the official price, including long-context tiers |

The key's list is the one that counts: a key created for **Only selected models** lists only those, and preview models appear only for keys with preview access.

## Ids and aliases

- Canonical ids are `author/model`, lowercase, such as `anthropic/claude-sonnet-5-5`.
- Anthropic's own names are aliases of the same model: the id without the `anthropic/` prefix, with the version written with `-` or `.`. The catalog's `aliases` field lists exactly which names work.
- Names are not case-sensitive. Responses echo the name you sent.
- Bedrock ids (`us.anthropic.…`, `global.anthropic.…`) and other providers' names are not accepted.

## Choosing

1. Use the model the user asks for, if it is in the key's list.
2. Otherwise read `GET https://api.router.ezat.io/api/models` and pick by the task: `capabilities` (`vision`, `tools`, `thinking`, `promptCaching`), `contextWindow`, `maxOutputTokens` and price. Prefer `status: "ACTIVE"`; `DEPRECATED` models still work but are on their way out. Skip entries with `available: false`.
3. Tell the user which model you picked and why, in one line.

Claude Code needs no model setting: its own model names are aliases. Set `ANTHROPIC_MODEL` only to force a specific id.

## Unknown model

`404 not_found_error` with `model: <name>` (OpenAI format: `404 model_not_found`) means the name isn't served to this key: a typo, a retired model, a preview model, or a name that isn't an alias. `403 permission_error` (`This API key is not allowed to use …`) means the key is limited to other models. Re-read the key's list and use an id from it.

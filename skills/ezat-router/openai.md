# OpenAI SDKs and OpenAI-format tools

For anything that speaks OpenAI's Chat Completions format: the `openai` SDKs, Cursor, Cline, LangChain's OpenAI client, and tools with an "OpenAI-compatible" or "custom base URL" setting.

| Setting | Value |
| --- | --- |
| Base URL | `https://api.router.ezat.io/openai/v1` (exactly: with `/openai/v1`) |
| API key | the EZAT Router key (sent as `Authorization: Bearer`) |
| Model | an id from `GET https://api.router.ezat.io/openai/v1/models`, such as `anthropic/claude-sonnet-5-5` |

## SDKs

Python:

```python
import os
from openai import OpenAI

client = OpenAI(base_url="https://api.router.ezat.io/openai/v1", api_key=os.environ["EZAT_API_KEY"])
completion = client.chat.completions.create(
    model="<model id>",
    max_tokens=16,
    messages=[{"role": "user", "content": "Reply with OK."}],
)
print(completion.choices[0].message.content)
```

TypeScript:

```ts
import OpenAI from "openai";

const client = new OpenAI({ baseURL: "https://api.router.ezat.io/openai/v1", apiKey: process.env.EZAT_API_KEY });
const completion = await client.chat.completions.create({
  model: "<model id>",
  max_tokens: 16,
  messages: [{ role: "user", content: "Reply with OK." }],
});
console.log(completion.choices[0].message.content);
```

The SDKs also read `OPENAI_BASE_URL` and `OPENAI_API_KEY` from the environment, but explicit constructor arguments win. Only change those variables if the user wants every OpenAI client on the machine to use EZAT Router.

## Tools

1. Find the tool's OpenAI or custom-provider settings: base URL, API key, model name.
2. Set the base URL to `https://api.router.ezat.io/openai/v1` and the model to an id from `GET https://api.router.ezat.io/openai/v1/models`. Ask the user to paste the key into the tool's own settings field themselves, or point the tool at `EZAT_API_KEY` if it reads environment variables.
3. Make one request in the tool and check it on `https://router.ezat.io/usage`.

## What works

- `POST https://api.router.ezat.io/openai/v1/chat/completions`, streaming (`stream: true`, and `stream_options.include_usage` for a final usage chunk) and not.
- `GET https://api.router.ezat.io/openai/v1/models` and `GET https://api.router.ezat.io/openai/v1/models/<id>`, with the same ids and aliases as the Anthropic API.
- `tools` and `tool_choice`, images (`image_url`: data URLs or http(s) URLs), PDFs as `file.file_data`, `response_format` with `json_schema`, `reasoning_effort`, `stop`.
- `max_tokens` or `max_completion_tokens`. Without either, the default is 32,000, capped at the model's output limit.
- Prompt caching is automatic. Send `x-ezat-cache: off` to turn it off for a request. `prompt_cache_key` keeps a conversation on the same capacity so its cache is reused.

## What doesn't

- Refused with `400`: `n` above 1, `logprobs` and `top_logprobs`, a non-empty `logit_bias`, audio input or output, `web_search_options`, the legacy `functions` API, `response_format` types other than `json_schema` and `text`.
- Dropped silently: `temperature` and `top_p` on models that don't accept them, and fields with no effect (`user`, `seed`, penalties, `store`, `metadata`, …).
- Not available: the Responses API, embeddings, audio, image generation.

## Errors in this format

Same HTTP statuses as the Anthropic API, in OpenAI's `{"error": {"message", "type", "param", "code"}}` shape, except:

| Anthropic API | OpenAI-compatible API |
| --- | --- |
| `529 overloaded_error` | `503`, code `overloaded` |
| `402 billing_error` | `402`, type `insufficient_quota` |
| `404 not_found_error` for a model | `404`, code `model_not_found` |
| `401 authentication_error` | `401`, code `invalid_api_key` |

A failure in the middle of a stream arrives as a `data:` chunk with an `error` object. More in `errors.md`.

Full pages: `https://docs.router.ezat.io/openai/base-url.md`, `https://docs.router.ezat.io/openai/chat-completions.md`, `https://docs.router.ezat.io/tools/openai-tools.md`.

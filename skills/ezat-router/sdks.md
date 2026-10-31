# Anthropic SDKs and curl

The Anthropic SDKs work unchanged. Set the base URL to `https://api.router.ezat.io` (no `/v1`) and pass the EZAT Router key as the API key. Read the key from the environment, on the server only.

## Python

```bash
pip install anthropic
```

```python
import os
import anthropic

client = anthropic.Anthropic(base_url="https://api.router.ezat.io", api_key=os.environ["EZAT_API_KEY"])
message = client.messages.create(
    model="<model id>",
    max_tokens=16,
    messages=[{"role": "user", "content": "Reply with OK."}],
)
print("".join(block.text for block in message.content if block.type == "text"))
```

Streaming: `with client.messages.stream(...) as stream: for text in stream.text_stream: ...`. Use streaming for long outputs: a request without streaming that runs over 590 seconds ends with `504 timeout_error`.

## TypeScript

```bash
npm install @anthropic-ai/sdk
```

```ts
import Anthropic from "@anthropic-ai/sdk";

const client = new Anthropic({ baseURL: "https://api.router.ezat.io", apiKey: process.env.EZAT_API_KEY });
const message = await client.messages.create({
  model: "<model id>",
  max_tokens: 16,
  messages: [{ role: "user", content: "Reply with OK." }],
});
for (const block of message.content) if (block.type === "text") console.log(block.text);
```

## curl

```bash
curl --fail-with-body -sS https://api.router.ezat.io/v1/messages \
  -H "x-api-key: $EZAT_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{"model": "<model id>", "max_tokens": 16, "messages": [{"role": "user", "content": "Reply with OK."}]}'
```

Add `"stream": true` for Server-Sent Events in Anthropic's format (`message_start`, `content_block_delta`, `message_delta`, `message_stop`, plus `ping` after 15 seconds of silence).

## Endpoints

| Endpoint | Notes |
| --- | --- |
| `POST /v1/messages` | Streaming and not. `model`, `max_tokens` and `messages` are required; every other field goes to the model unchanged. |
| `POST /v1/messages/count_tokens` | Free and not billed; counts toward the key's requests per minute. |
| `GET /v1/models` | The models this key can call. |

## Details that matter

- `anthropic-version` and `anthropic-beta` headers are accepted; beta features the model doesn't serve fail with `400`.
- The `model` in responses is exactly the name you sent.
- Every response has an `x-request-id` header; requests that passed every check and went to a model also have `request-id`. Log them, never the key.
- Prompt caching works as in Anthropic's API (`cache_control` breakpoints, 5-minute and 1-hour). Requests of one conversation stay on the same capacity so the cache is reused. To group requests yourself, send the same `x-ezat-session` header value on each.
- Not available: Message Batches, the Files API, server tools that run on Anthropic's side (web search, code execution), the MCP connector.
- The SDKs also read `ANTHROPIC_API_KEY` and `ANTHROPIC_BASE_URL` from the environment; explicit constructor arguments win. If both `ANTHROPIC_API_KEY` and `ANTHROPIC_AUTH_TOKEN` are set, the SDK sends both headers and EZAT Router uses `x-api-key`.

Full pages: `https://docs.router.ezat.io/anthropic/messages.md`, `https://docs.router.ezat.io/examples/python.md`, `https://docs.router.ezat.io/examples/node.md`, `https://docs.router.ezat.io/examples/curl.md`, `https://docs.router.ezat.io/guides/prompt-caching.md`.

# Errors, retries and support

## Shapes

- Anthropic format (`/v1/…`): `{"type": "error", "error": {"type": "<type>", "message": "…"}, "request_id": "…"}`.
- OpenAI format: `{"error": {"message": "…", "type": "…", "param": …, "code": "…"}}`, same HTTP status except `529` becomes `503`.
- A `404` with a `statusCode` field (`{"statusCode": 404, "message": "Cannot POST …"}`) is not an API error: the URL path is wrong. Fix the base URL (`quickstart.md`).

## By status

| Status | Type (OpenAI type or code) | Common messages and causes | Retry? |
| --- | --- | --- | --- |
| 400 | `invalid_request_error` | A required field is missing (`model`, `max_tokens`, `messages`), the body isn't valid JSON, or the model rejected a field or value; on the OpenAI API, a refused field (`openai.md`) | No. Fix the request. |
| 401 | `authentication_error` (`invalid_api_key`) | `Missing API key…`, `Invalid API key`, `This API key has been revoked`, `This API key has expired` | No. Check the key (`quickstart.md`). |
| 402 | `billing_error` (`insufficient_quota`) | `Your credit balance is too low…` (no credit), or `This API key reached its … spend limit …` | No. The user adds credit or uses another key. |
| 403 | `permission_error` | `This API key is not allowed to use …` (key's model list), `This organization is suspended…` | No. |
| 404 | `not_found_error` (`model_not_found`) | `model: <name>`: not a served id or alias for this key | No. Pick from `GET /v1/models`. |
| 413 | `request_too_large` | Body over 32 MB | No. Send less. |
| 429 | `rate_limit_error` (`rate_limit_exceeded`) | `This API key exceeded its requests-per-minute limit` (`retry-after: 30`), or `Rate limited upstream…` (`retry-after: 10`) | Yes, after `retry-after` |
| 500 | `api_error` (`server_error`) | Upstream or internal error | Yes, with backoff; if it persists, contact support |
| 504 | `timeout_error` (`server_error`, code `timeout`) | A request without streaming ran over 590 seconds | Yes, with `"stream": true` |
| 529 | `overloaded_error` (OpenAI: `503`, code `overloaded`) | No capacity for the model right now (`retry-after: 5`) | Yes, after `retry-after`; or another model |

## Retrying

1. Retry only `429`, `500`, `504` (with streaming) and `529`/`503`.
2. Wait at least the `retry-after` seconds, then back off exponentially with jitter. Give up after a few tries and tell the user.
3. The Anthropic and OpenAI SDKs already retry these by default; don't wrap them in a second retry loop that multiplies attempts.
4. EZAT Router already fails over between capacity before answering, but not once a stream has started.

## Rate limits

- Each key: 1,000 requests per minute by default, counted per calendar minute, including `POST /v1/messages/count_tokens`. Not a dashboard setting.
- Spend limits per key (day, month, total) answer with `402`, not `429` (`billing.md`).

## Streams

- After the response has started (status `200`), a failure arrives as an event: `event: error` with an `error` object (Anthropic format), or a `data:` chunk with an `error` object (OpenAI format). It isn't charged. Retry the whole request.
- Keep-alives arrive after 15 seconds without output (`event: ping`, or an SSE comment). Keep client read timeouts well above that.

## Request ids and support

- Every response has `x-request-id`. Requests that passed every check and went to a model also have `request-id`. Anthropic-format error bodies repeat `x-request-id` as `request_id`.
- For support, the user emails support@ezat.io with the `request-id` (or `x-request-id`), the time, the endpoint, the status and the message. Never include the key.

Full pages: `https://docs.router.ezat.io/reference/errors.md`, `https://docs.router.ezat.io/reference/rate-limits.md`, `https://docs.router.ezat.io/troubleshooting.md`.

# Quickstart: check the key, send a first request

## Pick the format

| The client speaks | Base URL | Endpoint |
| --- | --- | --- |
| Anthropic Messages (Claude Code, Anthropic SDKs) | `https://api.router.ezat.io` | `POST /v1/messages` |
| OpenAI Chat Completions (OpenAI SDKs, Cursor, Cline, LangChain's OpenAI client) | `https://api.router.ezat.io/openai/v1` | `POST /chat/completions` |
| Your own HTTP code | `https://api.router.ezat.io` (preferred: no translation) | `POST /v1/messages` |

Both take the same key and bill the same balance.

## Steps

1. Ask the user which environment variable holds their key (default `EZAT_API_KEY`). Don't ask for the key itself. If they have none: they create it at `https://router.ezat.io/keys` (shown once) and run `export EZAT_API_KEY="ezat_sk_…"` themselves.
2. Check it's set without revealing it:

   ```bash
   printf 'length %s, prefix %s\n' "${#EZAT_API_KEY}" "${EZAT_API_KEY:0:8}"
   ```

   Expect `length 48, prefix ezat_sk_`. Anything else: quotes, spaces, a line break or a cut-off copy. Ask the user to export it again.
3. Validate the key (free, not billed):

   ```bash
   curl --fail-with-body -sS -o /dev/null -w '%{http_code}\n' https://api.router.ezat.io/v1/models \
     -H "x-api-key: $EZAT_API_KEY"
   ```

   `200`: the key works. `401`: missing, mistyped, expired or revoked; stop and tell the user. `403`: the organization is suspended; the user emails support@ezat.io.
4. Pick a model id from `GET https://api.router.ezat.io/v1/models` (see `models.md`), or use the one the user asked for.
5. Send one tiny request:

   ```bash
   curl --fail-with-body -sS https://api.router.ezat.io/v1/messages \
     -H "x-api-key: $EZAT_API_KEY" \
     -H "anthropic-version: 2023-06-01" \
     -H "content-type: application/json" \
     -d '{"model": "<model id>", "max_tokens": 16, "messages": [{"role": "user", "content": "Reply with OK."}]}'
   ```

   `200` with a `content` array: done. `402`: no credit, or the key hit its spend limit; tell the user to add credit at `https://router.ezat.io/billing` and stop. Other errors: `errors.md`.
6. Configure the actual tool: `claude-code.md`, `openai.md` or `sdks.md`.
7. Confirm the request shows on `https://router.ezat.io/usage` (within a few seconds), if the user wants to check.

## Same check in the OpenAI format

```bash
curl --fail-with-body -sS https://api.router.ezat.io/openai/v1/chat/completions \
  -H "Authorization: Bearer $EZAT_API_KEY" \
  -H "content-type: application/json" \
  -d '{"model": "<model id>", "max_tokens": 16, "messages": [{"role": "user", "content": "Reply with OK."}]}'
```

## Base URL mistakes

- `https://api.router.ezat.io/v1` as an Anthropic base URL gives `404 Cannot POST /v1/v1/messages`.
- `https://api.router.ezat.io` or `https://api.router.ezat.io/v1` as an OpenAI base URL gives `404` (`Cannot POST /v1/chat/completions` or `Not Found`). Use `https://api.router.ezat.io/openai/v1`.

Full page: `https://docs.router.ezat.io/quickstart.md`.

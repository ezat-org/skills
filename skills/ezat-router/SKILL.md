---
name: ezat-router
description: "Use when setting up or debugging any tool, SDK or code that calls AI models through EZAT Router (ezat_sk_ API keys): Claude Code, the Anthropic or OpenAI SDKs, Cursor, Cline, LangChain or plain HTTP. Covers API keys, base URLs, choosing models, balance and billing, and errors such as 401, 402, 404, 429 and 529."
metadata:
  version: "1.2.0"
---

# EZAT Router

EZAT Router is an AI model API: one `ezat_sk_…` key and one prepaid balance per organization, billed per token.
It serves an Anthropic-compatible API (`/v1/messages`) and an OpenAI-compatible one (Chat Completions).
Models run on Amazon Bedrock. Users manage keys and credit in the dashboard; you never do.

## Fixed facts

| What | Value |
| --- | --- |
| Anthropic-format base URL | `https://api.router.ezat.io` (clients add `/v1/messages`; never add `/v1` yourself) |
| OpenAI-format base URL | `https://api.router.ezat.io/openai/v1` |
| Auth header (both APIs) | `x-api-key: <key>` or `Authorization: Bearer <key>`. If both are sent, `x-api-key` wins. |
| Key format | `ezat_sk_` + 40 letters and digits = 48 characters |
| Key variable | `EZAT_API_KEY`, unless the user names another |
| Models | `GET https://api.router.ezat.io/v1/models` with the key. Ids are `author/model`. |
| Dashboard | `https://router.ezat.io`: keys at `https://router.ezat.io/keys`, credit at `https://router.ezat.io/billing`, usage at `https://router.ezat.io/usage` |
| Support | support@ezat.io, quoting the `request-id` response header |

## Rules

1. Read the key from `EZAT_API_KEY` (or the variable the user names) at runtime.
2. Never ask the user to paste a key into the chat. If there is no key, tell them to create one at `https://router.ezat.io/keys` and export it in their own terminal.
3. Never print, echo, log or commit a key. To check one, print only its length and first 8 characters.
4. Never put a key in frontend, browser or mobile code, in a URL, or in a committed file (a project's `.claude/settings.json`, a tracked `.env`, source code).
5. Never sign up, pay, top up, or create or revoke keys for the user.
6. Never hard-code prices or model lists. Read them live (see `models.md`).
7. Ask before changing global config the user didn't mention, and don't break their existing Anthropic or OpenAI setup for other work.
8. Test with tiny requests (`max_tokens` of 16 or so). Don't retry `400`, `401`, `402`, `403` or `404` in a loop.

## Load only what you need

| Load | When |
| --- | --- |
| `quickstart.md` | First request, checking a key, which base URL and format to use |
| `claude-code.md` | Pointing Claude Code (or another Anthropic-format tool) at EZAT Router |
| `openai.md` | OpenAI SDKs, Cursor, Cline, LangChain and other tools that take an OpenAI base URL |
| `sdks.md` | Writing code with the Anthropic SDK (Python, TypeScript) or curl |
| `models.md` | Choosing a model, listing models, aliases, prices |
| `billing.md` | Balance, credit, what a request costs, spend limits, interrupted streams |
| `errors.md` | Any error status or type, retries, rate limits, `request-id`, contacting support |

## Live docs

This skill is a copy and doesn't update itself. The live docs are newer:

- `https://docs.router.ezat.io/llms.txt`: index of every page. Read it first when this skill doesn't cover something.
- `https://docs.router.ezat.io/<page>.md`: any page as Markdown, for example `https://docs.router.ezat.io/troubleshooting.md`.
- `https://docs.router.ezat.io/llms-full.txt`: every page in one file.
- `https://docs.router.ezat.io/openapi.yaml`: the API spec.
- Latest copy of this skill: `https://docs.router.ezat.io/skill/ezat-router.zip` (this is version 1.2.0). The same skill is in the GitHub repository `ezat-org/skills`: Claude Code installs it as the plugin `ezat-router@ezat`, and other agents with `npx skills add ezat-org/skills`.

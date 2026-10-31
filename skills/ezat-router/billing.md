# Balance, credit and what requests cost

## The model

- Prepaid. Each organization has one balance in USD. Every API key of the organization draws on it.
- The user adds credit by card (Stripe) at `https://router.ezat.io/billing`, as an owner, admin or billing member. Credit shows a few seconds after payment and doesn't expire. You never pay or top up for the user.
- Each request is charged per token at the model's price: input, output, cache writes (5-minute and 1-hour) and cache reads, with long-context prices on models that have them. Prices: `https://docs.router.ezat.io/pricing.md`. Never quote a price from memory.
- The same conversation costs the same through the Anthropic and the OpenAI-compatible API.
- `POST /v1/messages/count_tokens` and `GET /v1/models` are free.

## When a request is charged

| Outcome | Charged |
| --- | --- |
| Refused before reaching a model (bad key, unknown model, limits, no credit) | Nothing |
| Succeeded | Its tokens, from the response's `usage` |
| Failed at the model, including a stream that ended with an error event | Nothing |
| Client disconnected mid-stream | Input and cache tokens exactly, output generated so far (estimated if the stream ended before the final count) |

The balance is checked before each request and charged when it finishes. A request that starts with a positive balance always finishes, so the balance can end slightly below zero; the next top-up covers it.

## Limits

- Balance at or below zero: every request fails with `402 billing_error` (`Your credit balance is too low…`; OpenAI format `insufficient_quota`) until credit is added. Don't retry; tell the user.
- A key can have a spend limit per UTC day, per UTC calendar month, or in total. When reached: `402 billing_error` naming the limit. Day and month limits reset by themselves; a total limit needs a new key. Limits can't be edited after the key is created.
- Credit belongs to one organization. If the user added credit and still gets `402`, ask whether it went to the organization the key belongs to.

## Where the user sees it

- `https://router.ezat.io/usage`: requests, tokens and cost by day, model and key, plus each request with its status.
- `https://router.ezat.io/billing`: balance, payments, credit activity, and an optional low balance email alert.

Full page: `https://docs.router.ezat.io/billing.md`.

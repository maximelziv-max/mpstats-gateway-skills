---
name: mpstats-gateway
description: Use MPSTATS Data Gateway from AI agents through the internal quota-aware proxy instead of calling MPSTATS directly.
---

# MPSTATS Data Gateway

Use this skill whenever a user asks an AI agent to work with MPSTATS data
through MPSTATS Data Gateway.

The gateway is the only allowed network target. Do not call `mpstats.io`,
public Wildberries APIs, or any older/direct `mpstats` skill unless the
operator explicitly asks for a separate public-data check. Do not ask the user
for the MPSTATS master token.

## Required Configuration

Read these values from the agent environment or ask the operator to provide
them outside chat/logs:

```text
MPSTATS_GATEWAY_BASE_URL=https://mpstats.market-master.site
MPSTATS_GATEWAY_API_KEY=<authorization token from the gateway>
```

`MPSTATS_GATEWAY_BASE_URL` is the root Gateway URL. Do not add
`/api/v1/mpstats` to this variable; request examples below add the proxy prefix.

The API key is the user's gateway authorization token. It is not the MPSTATS
master token.

## Before Any Data Request

1. Read `references/endpoints.md`.
2. Read `references/request-patterns.md`.
3. Call `GET /status` on `MPSTATS_GATEWAY_BASE_URL`.
4. Call `GET /api/v1/quota` with `X-Api-Key`.
5. Continue only when gateway status is `active`, user status is `active`, and
   remaining quota is greater than `0`.

## Request Rules

- Build data URLs exactly as:
  `MPSTATS_GATEWAY_BASE_URL + "/api/v1/mpstats/" + whitelisted_path`.
- Never call `/api/analytics/...`, `/analytics/...`, `/api/v1/analytics/...`,
  or `mpstats.io` directly.
- Never invoke an installed direct `mpstats` skill, direct MPSTATS scripts, or
  public WB/Card/WBBasket API helpers for MPSTATS Gateway tasks. If a needed
  data point is not available through the Gateway whitelist, stop and report
  the missing endpoint.
- Use `POST` for table/list endpoints. Use `GET` only for whitelisted SKU
  detail endpoints in `references/endpoints.md`.
- Use only `/api/v1/mpstats/...` paths listed in `references/endpoints.md`.
- Use the internal header `X-Api-Key: <gateway authorization token>`.
- Add an `Idempotency-Key` for every retryable request.
- Do not send or store MPSTATS master tokens.
- Do not invent paths. If the needed path is not whitelisted, ask the operator
  to add it to the gateway backlog.
- Treat `safe_mode`, `pool_exhausted`, `frozen`, `blocked`, and `inactive` as
  stop conditions.
- Expect agent accounts to run in `strict_1_to_1` mode unless the operator
  explicitly switches them to an economy mode. In strict mode, a normal data
  request is sent to MPSTATS once and cache/deduplication should not be assumed.
- The agent sends requests only to the Gateway. The Gateway injects the MPSTATS
  token server-side and sends the upstream MPSTATS request through the
  configured outbound proxy. The agent must not configure or use the MPSTATS
  proxy directly.

## Response Handling

Always inspect these response headers when present:

- `X-Request-ID`
- `X-Consumed-Quota`
- `X-MPSTATS-Calls`
- `X-Total-Consumed`
- `X-Remaining-Quota`
- `X-User-Status`
- `X-Cache`

Do not maintain a separate quota counter. Trust `/api/v1/quota` and the gateway
headers.

`X-Consumed-Quota` is the billable successful MPSTATS result count. `X-MPSTATS-Calls`
is the number of upstream MPSTATS attempts made for this gateway request.
Retries or processing responses may make these values differ outside strict
mode.

## Minimal Agent Flow

```bash
curl "$MPSTATS_GATEWAY_BASE_URL/status"

curl "$MPSTATS_GATEWAY_BASE_URL/api/v1/quota" \
  -H "X-Api-Key: $MPSTATS_GATEWAY_API_KEY"
```

Then call a whitelisted endpoint:

```bash
curl -X POST "$MPSTATS_GATEWAY_BASE_URL/api/v1/mpstats/analytics/v1/wb/items?ids=123456" \
  -H "X-Api-Key: $MPSTATS_GATEWAY_API_KEY" \
  -H "Idempotency-Key: wb-item-123456" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## Safety

If an error happens, report the status code, `X-Request-ID`, `X-Cache`, and
remaining quota. Never print secrets.

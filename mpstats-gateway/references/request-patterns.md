# Request Patterns

## Gateway Target

```text
MPSTATS_GATEWAY_BASE_URL=https://mpstats.market-master.site
```

Use this as the root Gateway URL. Data requests add the proxy prefix
`/api/v1/mpstats/...`.

Agents send requests only to the Gateway. The Gateway sends upstream requests
to MPSTATS through its configured outbound proxy and never exposes the MPSTATS
master token to the agent.

Do not guess alternate routes. These are invalid and should not be retried:

- `GET /api/v1/mpstats/...` except the whitelisted SKU detail endpoints
- `/api/analytics/...`
- `/analytics/...`
- `/api/v1/analytics/...`
- direct `https://mpstats.io/api/...`
- direct MPSTATS scripts from any older `mpstats` skill
- public Wildberries/Card/WBBasket requests as a substitute for missing Gateway
  endpoints

## Standard Headers

```text
X-Api-Key: <gateway authorization token>
Idempotency-Key: <stable id for retryable request>
Content-Type: application/json
```

Use `Idempotency-Key` for retries so a repeated final `200` response is not
counted twice.

## Category Request

```bash
curl -X POST "$MPSTATS_GATEWAY_BASE_URL/api/v1/mpstats/analytics/v1/wb/category/items?d1=2026-01-01&d2=2026-01-31&path=0" \
  -H "X-Api-Key: $MPSTATS_GATEWAY_API_KEY" \
  -H "Idempotency-Key: category-0-2026-01" \
  -H "Content-Type: application/json" \
  -d '{"startRow":0,"endRow":100,"filterModel":{},"sortModel":[]}'
```

Keep `endRow - startRow <= 500`. Larger pages are rejected by the Gateway with
zero newly consumed quota.

For a compact category trend report, prefer:

1. `POST category/by_date`
2. `POST category/trends`
3. `POST category/price_segmentation`
4. optionally `POST category/keywords` or `POST category/brands`

Use `category/compare` only when the task explicitly needs two-period
comparison.

## Item Request

```bash
curl -X POST "$MPSTATS_GATEWAY_BASE_URL/api/v1/mpstats/analytics/v1/wb/items?ids=123456" \
  -H "X-Api-Key: $MPSTATS_GATEWAY_API_KEY" \
  -H "Idempotency-Key: item-123456" \
  -H "Content-Type: application/json" \
  -d '{}'
```

## SKU Full Request

```bash
curl "$MPSTATS_GATEWAY_BASE_URL/api/v1/mpstats/analytics/v1/wb/items/331755125/full?d1=2026-05-27&d2=2026-06-25" \
  -H "X-Api-Key: $MPSTATS_GATEWAY_API_KEY" \
  -H "Idempotency-Key: sku-331755125-full-2026-05-27-2026-06-25"
```

For a compact SKU report, prefer at most:

1. `GET items/{sku}/full`
2. `GET items/{sku}/by_period`
3. optionally `POST items/{sku}/keywords`

## Stop Conditions

Stop and ask the operator when:

- `/status` returns `gateway_status` other than `active`.
- `/api/v1/quota` returns user status `frozen`, `blocked`, or `inactive`.
- Remaining quota is `0`.
- The gateway returns `401`, `403`, `409`, or `429`.
- The needed MPSTATS path is not whitelisted.

## Counting Rules

The gateway counts quota only after a final successful MPSTATS `200` response.
Cache hits, idempotent replays, rejected whitelist paths, pre-flight denials,
`202`, `401`, `429`, `4xx`, `5xx`, and timeouts report
`X-Consumed-Quota: 0`.

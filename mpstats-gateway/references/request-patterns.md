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

## Cache Discipline

Gateway caches successful `200` data responses for 30 days. Cache is active for
all account request modes, including `strict_1_to_1`.

For repeated analysis, reuse the exact same endpoint path, query parameters,
and JSON body. Query and JSON field ordering are normalized, but changing dates,
path, page range, filters, FBS flag, SKU, or report type creates a new cache key
and can spend quota again.

On cache reuse, expect:

```text
X-Cache: HIT
X-Consumed-Quota: 0
X-MPSTATS-Calls: 0
```

## Quota Budgeting

Before data requests:

1. Call `GET /api/v1/quota`.
2. Call `GET /api/v1/quota/pricing`.
3. Estimate the planned calls and total endpoint cost.
4. Stop and ask the operator if remaining quota is lower than the planned cost.

Read these response headers after every request:

- `X-Quota-Cost`: expected endpoint cost.
- `X-Consumed-Quota`: newly charged quota after a final successful `200`.
- `X-Remaining-Quota`: remaining account quota.
- `X-MPSTATS-Calls`: upstream attempts, not billable units.

Current high-cost production rules:

| Endpoint key | Cost |
|---|---:|
| `analytics/v1/wb/category/items` | 30 |
| `analytics/v1/wb/category/categories` | 20 |
| Other `analytics/v1/wb/category/...` analytics reports | 20 |
| `analytics/v1/wb/items` | 3 |
| `analytics/v1/wb/items/{sku}/full` | 3 |
| `analytics/v1/wb/items/{sku}/by_period` | 3 |
| `analytics/v1/wb/items/{sku}/keywords` | 3 |

For niche/category analysis, do not run broad exploratory category calls in a
loop. Use one rubricator lookup, choose the exact path, then run the minimum
heavy reports needed for the user's question. Do not repeat the same failed or
empty query with small variations unless the operator approves the extra cost.

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

For a full niche report, explain the expected cost before calling
`category/items` or `category/categories`. These endpoints are intentionally
priced as heavy calls.

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

Do not add balance, comments, FAQ, photo history, and keyword reports unless
the user asks for that detail; each extra report may consume additional quota.

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

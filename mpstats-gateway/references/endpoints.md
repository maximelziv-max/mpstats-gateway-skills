# Whitelisted Gateway Endpoints

Base URL:

```text
MPSTATS_GATEWAY_BASE_URL=https://mpstats.market-master.site
```

Proxy prefix:

```text
/api/v1/mpstats/
```

Hard rule: every data request must start with
`https://mpstats.market-master.site/api/v1/mpstats/`.
Use `POST` for table/list endpoints and `GET` only for the SKU detail endpoints
listed below.
Do not call `/api/analytics/...`, `/analytics/...`, `/api/v1/analytics/...`,
or any MPSTATS endpoint directly.

Allowed Wildberries v1 paths:

| Purpose | Gateway path | Method |
|---|---|---|
| Category rubricator | `/api/v1/mpstats/analytics/v1/wb/category/list` | POST |
| Category or niche analysis | `/api/v1/mpstats/analytics/v1/wb/category/items` | POST |
| Subcategory analytics | `/api/v1/mpstats/analytics/v1/wb/category/categories` | POST |
| Category brands | `/api/v1/mpstats/analytics/v1/wb/category/brands` | POST |
| Category sellers | `/api/v1/mpstats/analytics/v1/wb/category/sellers` | POST |
| Category trends | `/api/v1/mpstats/analytics/v1/wb/category/trends` | POST |
| Category daily metrics | `/api/v1/mpstats/analytics/v1/wb/category/by_date` | POST |
| Category price segmentation | `/api/v1/mpstats/analytics/v1/wb/category/price_segmentation` | POST |
| Category keywords | `/api/v1/mpstats/analytics/v1/wb/category/keywords` | POST |
| Category period compare | `/api/v1/mpstats/analytics/v1/wb/category/compare` | POST |
| Category subjects | `/api/v1/mpstats/analytics/v1/wb/category/subjects` | POST |
| Category warehouses | `/api/v1/mpstats/analytics/v1/wb/category/warehouses` | POST |
| Brand analytics | `/api/v1/mpstats/analytics/v1/wb/brand/items` | POST |
| Seller analytics | `/api/v1/mpstats/analytics/v1/wb/seller/items` | POST |
| Item or SKU analytics | `/api/v1/mpstats/analytics/v1/wb/items` | POST |

Allowed Wildberries SKU detail paths:

| Purpose | Gateway path | Method |
|---|---|---|
| SKU info | `/api/v1/mpstats/analytics/v1/wb/items/{sku}` | GET |
| SKU full sales analytics | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/full` | GET |
| SKU daily sales and stock | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/by_period` | GET |
| SKU balance by stores | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/balance/stores` | GET |
| SKU balance by sizes | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/balance/sizes` | GET |
| SKU balance by stores and sizes | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/balance/stores_and_sizes` | GET |
| SKU balance by colors | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/balance/colors` | GET |
| SKU sales by stores | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/sales/stores` | GET |
| SKU sales by sizes | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/sales/sizes` | GET |
| SKU sales heatmap | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/sales/heatmap` | GET |
| SKU stores | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/stores` | GET |
| SKU search stats | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/search_stats` | GET |
| SKU keywords | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/keywords` | POST |
| SKU hourly keywords | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/keywords/hourly` | POST |
| SKU comments | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/comments` | GET |
| SKU FAQ | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/faq` | GET |
| SKU photo history | `/api/v1/mpstats/analytics/v1/wb/items/{sku}/photos_history` | GET |

`{sku}` must be a numeric Wildberries item id.

Legacy MPSTATS-style aliases are also accepted by the Gateway and translated
server-side to the modern whitelist:

| Legacy Gateway path | Canonical path |
|---|---|
| `/api/v1/mpstats/wb/get/category` | `/api/v1/mpstats/analytics/v1/wb/category/items` |
| `/api/v1/mpstats/wb/get/subcategory` | `/api/v1/mpstats/analytics/v1/wb/category/categories` |
| `/api/v1/mpstats/wb/get/brand` | `/api/v1/mpstats/analytics/v1/wb/brand/items` |
| `/api/v1/mpstats/wb/get/seller` | `/api/v1/mpstats/analytics/v1/wb/seller/items` |
| `/api/v1/mpstats/wb/get/item` | `/api/v1/mpstats/analytics/v1/wb/items` |

The gateway intentionally rejects arbitrary MPSTATS paths. A rejected path
returns zero newly consumed quota.

The agent calls only the Gateway paths above. The Gateway handles the real
MPSTATS request, server-side token injection, quota accounting, and outbound
proxy routing.

## Health And Quota

```text
GET /status
GET /api/v1/ping
GET /api/v1/quota
```

`/api/v1/quota` requires `X-Api-Key`.

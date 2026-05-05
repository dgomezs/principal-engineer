# OpenAPI Search Endpoints (Stripe-style)

Search is separate from list. List endpoints return resources in default order with cursor pagination. Search endpoints support filtering, sorting, and use offset-based pagination with a bounded result window.

## When to use list vs search

| Concern | List (`list{Resources}`) | Search (`search{Resources}`) |
|---|---|---|
| Path | `GET /v1/{resources}` | `GET /v1/{resources}/search` |
| Pagination | Cursor-based (`starting_after` / `ending_before`) | Offset-based (`limit` + `page`) |
| Sort | Fixed — `created_at` desc only | Configurable via `sort` parameter |
| Filters | Simple equality filters only (e.g., `status`, `customer_id`) | Rich filters (ranges, text search, multiple fields) |
| Result window | Unbounded — can page through all results | Bounded — max 10,000 results (page * limit ≤ 10,000) |
| Total count | No (`has_more` only) | Yes (`total_count` returned) |

## Search request parameters

```yaml
# Pagination
SearchLimit:
  name: limit
  in: query
  schema:
    type: integer
    minimum: 1
    maximum: 100
    default: 10
  description: Number of items to return.

SearchPage:
  name: page
  in: query
  schema:
    type: integer
    minimum: 1
    default: 1
  description: Page number to return (1-indexed). page * limit must not exceed 10,000.

# Sorting
SearchSort:
  name: sort
  in: query
  schema:
    type: string
  description: >
    Field to sort by, prefixed with `-` for descending.
    Example: `amount` (ascending), `-created_at` (descending).
    Allowed fields are resource-specific (e.g., `amount`, `created_at`, `status`).
```

## Search response

```yaml
OrderSearchResult:
  type: object
  required: [object, data, total_count, url]
  properties:
    object:
      type: string
      enum: [search_result]
      description: "String representing the object's type. Always `search_result`."
    data:
      type: array
      items:
        $ref: "#/components/schemas/OrderResponse"
      description: The list of matching resources.
    total_count:
      type: integer
      description: Total number of results matching the query (capped at 10,000).
    has_more:
      type: boolean
      description: Whether there are more results beyond the current page.
    url:
      type: string
      description: The URL for accessing this search.
```

## Search filters

Filters use query parameters with typed suffixes for range operations:

```yaml
# Equality
?status=confirmed
?customer_id=cus_5kHb9lS0qMrOwY8nCdZxAt4U

# Ranges (suffix with _gt, _gte, _lt, _lte)
?amount_gte=1000&amount_lte=5000
?created_at_gte=1711036800

# Text search (keyword match)
?query=widget
```

## Example search endpoint

```yaml
/orders/search:
  get:
    operationId: searchOrders
    summary: Search orders
    description: >
      Search and filter orders with sorting and offset-based pagination.
      Result window is limited to 10,000 items.
    parameters:
      - $ref: "#/components/parameters/SearchLimit"
      - $ref: "#/components/parameters/SearchPage"
      - $ref: "#/components/parameters/SearchSort"
      - name: status
        in: query
        schema:
          $ref: "#/components/schemas/OrderStatus"
        description: Filter by order status.
      - name: customer_id
        in: query
        schema:
          $ref: "#/components/schemas/CustomerId"
        description: Filter by customer.
      - name: amount_gte
        in: query
        schema:
          type: integer
        description: Minimum amount (inclusive), in smallest currency unit.
      - name: amount_lte
        in: query
        schema:
          type: integer
        description: Maximum amount (inclusive), in smallest currency unit.
      - name: created_at_gte
        in: query
        schema:
          type: integer
          format: int64
        description: Minimum creation time (inclusive), Unix timestamp.
      - name: created_at_lte
        in: query
        schema:
          type: integer
          format: int64
        description: Maximum creation time (inclusive), Unix timestamp.
      - name: query
        in: query
        schema:
          type: string
        description: Keyword search across resource fields.
    responses:
      "200":
        description: Search results.
        content:
          application/json:
            schema:
              $ref: "#/components/schemas/OrderSearchResultResponse"
```

## Rules

- Search endpoints live at `GET /v1/{resources}/search`
- `operationId` follows `search{Resources}` pattern
- Offset-based pagination with `limit` + `page` (1-indexed)
- Result window capped at 10,000 (`page * limit ≤ 10,000`) — return 400 if exceeded
- `total_count` is included but capped at 10,000 (use `has_more` to indicate more exist)
- Sort via `sort` parameter: field name for ascending, `-field` for descending
- Default sort is `-created_at` (newest first) when no `sort` is specified
- Range filters use `_gt`, `_gte`, `_lt`, `_lte` suffixes
- Response `object` field is `search_result`, not `list`
- Each resource defines its own allowed sort fields and filter parameters
- Search endpoints are query handlers — they bypass the domain model and query persistence directly

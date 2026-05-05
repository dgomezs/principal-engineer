# OpenAPI Specification (Sync APIs)

OpenAPI 3.1 specs following Stripe API conventions. Contract-first — the spec drives code generation.

## General Structure

```yaml
openapi: "3.1.0"
info:
  title: Order Service API
  version: "1.0.0"
  description: Manages order lifecycle.
servers:
  - url: /v1
paths:
  # ...
components:
  schemas:
    # ...
  parameters:
    # ...
  responses:
    # ...
```

## Resource & URL Conventions (Stripe-style)

- Plural nouns, lowercase, hyphen-separated: `/v1/orders`, `/v1/order-items`
- Nested resources for strong ownership: `/v1/orders/{order_id}/items`
- URL path parameters use `snake_case`: `{order_id}`, not `{orderId}`
- Version prefix in the server URL (`/v1`), not in each path
- No verbs in URLs — use HTTP methods: `POST /v1/orders` not `POST /v1/create-order`
- No trailing slashes

## Prefixed IDs (Stripe-style)

Every resource type has a short prefix for its identifiers:

```yaml
OrderId:
  type: string
  pattern: "^ord_[a-zA-Z0-9]{26}$"
  example: "ord_2jFa8kR9pLqNvX7mBcYwZs3T"
  description: Unique identifier for the order.
```

Common prefixes: `ord_` (order), `itm_` (item), `pay_` (payment), `shp_` (shipment), `cus_` (customer).

## Operations

### CQRS Alignment

| HTTP Method | Purpose         | operationId pattern    | Example                |
|-------------|-----------------|------------------------|------------------------|
| POST        | Create/command  | `create{Resource}`     | `createOrder`          |
| GET (by id) | Query single    | `retrieve{Resource}`   | `retrieveOrder`        |
| GET (list)  | Query list      | `list{Resources}`      | `listOrders`           |
| GET (search)| Query with filters/sort | `search{Resources}` | `searchOrders`     |
| POST (action)| Command on resource | `{verb}{Resource}` | `cancelOrder`          |
| PATCH       | Partial update  | `update{Resource}`     | `updateOrder`          |
| DELETE      | Remove          | `delete{Resource}`     | Not used (soft delete) |

- `operationId` is **camelCase** — it drives generated method names
- Avoid PUT — use PATCH for partial updates (Stripe convention)
- Actions on resources use POST with a verb path: `POST /v1/orders/{order_id}/cancel`

## Request & Response Schemas

### Naming Convention

- Create request: `Create{Resource}Request` — `CreateOrderRequest`
- Update request: `Update{Resource}Request` — `UpdateOrderRequest`
- Response: `{Resource}Response` — `OrderResponse`
- List response: `{Resource}ListResponse` — `OrderListResponse`
- Search response: `{Resource}SearchResultResponse` — `OrderSearchResultResponse`

### Resource Object (Stripe-style)

Every resource includes an `object` discriminator field:

```yaml
OrderResponse:
  type: object
  required: [id, object, status, customer_id, items, amount, currency, created_at]
  properties:
    id:
      $ref: "#/components/schemas/OrderId"
    object:
      type: string
      enum: [order]
      description: "String representing the object's type. Always `order`."
    status:
      $ref: "#/components/schemas/OrderStatus"
    customer_id:
      $ref: "#/components/schemas/CustomerId"
    items:
      type: array
      items:
        $ref: "#/components/schemas/OrderItem"
    amount:
      type: integer
      description: Total amount in the smallest currency unit (e.g. cents).
    currency:
      type: string
      pattern: "^[a-z]{3}$"
      example: "usd"
      description: Three-letter ISO 4217 currency code, lowercase.
    metadata:
      $ref: "#/components/schemas/Metadata"
    created_at:
      type: integer
      format: int64
      description: Unix timestamp (seconds) when the resource was created.
```

### Monetary Amounts

- Always an integer in the **smallest currency unit** (cents for USD)
- Pair with a `currency` field (ISO 4217, lowercase)
- Never use floating point for money

### Timestamps

- Unix timestamps (seconds since epoch) as `integer` / `int64`
- Field names end in `_at`: `created_at`, `updated_at`, `cancelled_at`

### Metadata

Arbitrary key-value pairs for consumer use:

```yaml
Metadata:
  type: object
  additionalProperties:
    type: string
  description: Set of key-value pairs for storing additional information.
  maxProperties: 50
```

### Snake Case for All Fields

All schema property names use `snake_case` — aligns with Stripe convention and JSON best practices.

## Cursor-Based Pagination (Stripe-style)

```yaml
OrderList:
  type: object
  required: [object, data, has_more, url]
  properties:
    object:
      type: string
      enum: [list]
    data:
      type: array
      items:
        $ref: "#/components/schemas/Order"
    has_more:
      type: boolean
      description: Whether there are more items beyond this page.
    url:
      type: string
      description: The URL for accessing this list.

# Pagination parameters (reusable)
PaginationLimit:
  name: limit
  in: query
  schema:
    type: integer
    minimum: 1
    maximum: 100
    default: 10
  description: Number of items to return.

PaginationStartingAfter:
  name: starting_after
  in: query
  schema:
    type: string
  description: Cursor for forward pagination. Value is the ID of the last item in the previous page.

PaginationEndingBefore:
  name: ending_before
  in: query
  schema:
    type: string
  description: Cursor for backward pagination. Value is the ID of the first item in the previous page.
```

- No offset-based pagination — cursor-based only
- Default page size: 10, max: 100
- `has_more` boolean instead of total count
- List endpoints use the default sort order (`created_at` desc) — no custom sort fields

## Expandable Fields (Stripe-style)

```yaml
# Query parameter
Expand:
  name: expand
  in: query
  schema:
    type: array
    items:
      type: string
  style: deepObject
  description: "Fields to expand. Example: ?expand[]=customer&expand[]=items"
```

Resources reference related objects by ID by default. The `expand` parameter inlines the full object.

## Idempotency

```yaml
IdempotencyKey:
  name: Idempotency-Key
  in: header
  schema:
    type: string
    format: uuid
  description: Unique key to ensure exactly-once processing for POST requests.
```

- Required on all mutating operations (POST, PATCH, DELETE)
- Server returns cached response for duplicate keys

## Error Responses (Stripe-style)

```yaml
Error:
  type: object
  required: [error]
  properties:
    error:
      type: object
      required: [type, message]
      properties:
        type:
          type: string
          enum:
            - invalid_request_error
            - authentication_error
            - rate_limit_error
            - api_error
          description: The type of error.
        code:
          type: string
          description: Machine-readable error code.
          example: "resource_not_found"
        message:
          type: string
          description: Human-readable message.
        param:
          type: string
          description: The parameter related to the error, if applicable.
```

### HTTP Status Code Usage

| Status | When                                   | Error type                |
|--------|----------------------------------------|---------------------------|
| 200    | Success (retrieve, update, action)     | —                         |
| 201    | Resource created                       | —                         |
| 400    | Invalid parameters                     | `invalid_request_error`   |
| 401    | Missing or invalid authentication      | `authentication_error`    |
| 404    | Resource not found                     | `invalid_request_error`   |
| 409    | Conflict (idempotency, state)          | `invalid_request_error`   |
| 429    | Rate limited                           | `rate_limit_error`        |
| 500    | Internal server error                  | `api_error`               |

## Correlation

All responses include the `X-Correlation-Id` header:

```yaml
CorrelationIdHeader:
  name: X-Correlation-Id
  in: header
  schema:
    type: string
  description: Correlation ID for request tracing.
```

## Rules

- OpenAPI 3.1 only
- YAML format only
- All property names in `snake_case`
- All `operationId` values in `camelCase`
- URL path parameters in `snake_case`
- Every resource has an `id`, `object`, and `created_at` field
- Use prefixed IDs (`ord_`, `cus_`, etc.) — never raw UUIDs in the API
- Monetary amounts as integers in smallest currency unit + `currency` field
- Timestamps as Unix epoch seconds (int64), suffixed `_at`
- List endpoints: cursor-based pagination only — no offset, no total count, no custom sort
- Search endpoints: see `openapi-search.md`
- Error responses wrapped in `error` object with `type`, `message`, optional `code` and `param`
- Every schema property must have a `description`
- Reusable components go in `components/` (parameters, schemas, responses)
- No inline schema definitions in path items — always `$ref`

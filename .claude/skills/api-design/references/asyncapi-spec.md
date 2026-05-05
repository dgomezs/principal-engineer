# AsyncAPI Specification (Async Events)

AsyncAPI 3.0 specs for Kafka event contracts with Avro payloads. AsyncAPI is the human-readable contract; `.avsc` files are the source of truth for payload schemas and drive code generation.

## File Layout

```
src/main/resources/api/
├── asyncapi.yaml                   # Contract documentation (channels, operations, bindings)
└── avro/                           # Avro schemas (source of truth for payloads)
    ├── common/
    │   ├── EventHeaders.avsc
    │   └── Money.avsc
    ├── order/
    │   ├── OrderPlacedPayload.avsc
    │   ├── OrderConfirmedPayload.avsc
    │   ├── OrderCancelledPayload.avsc
    │   └── OrderItemData.avsc
    └── payment/
        └── PaymentProcessedPayload.avsc
```

## AsyncAPI Structure

```yaml
asyncapi: "3.0.0"
info:
  title: Order Service Events
  version: "1.0.0"
  description: Domain events published by the Order Service.
defaultContentType: avro/binary

channels:
  orderPlaced:
    address: order.events.placed
    messages:
      orderPlacedMessage:
        $ref: "#/components/messages/OrderPlacedEvent"
    bindings:
      kafka:
        topic: order.events.placed
        partitions: 6
        replicas: 3

operations:
  publishOrderPlaced:
    action: send
    channel:
      $ref: "#/channels/orderPlaced"
    messages:
      - $ref: "#/channels/orderPlaced/messages/orderPlacedMessage"

  consumeOrderPlaced:
    action: receive
    channel:
      $ref: "#/channels/orderPlaced"
    messages:
      - $ref: "#/channels/orderPlaced/messages/orderPlacedMessage"

components:
  messages:
    OrderPlacedEvent:
      name: OrderPlacedEvent
      title: Order Placed
      description: Published when a new order is successfully placed.
      contentType: avro/binary
      headers:
        $ref: "#/components/schemas/EventHeaders"
      payload:
        schemaFormat: "application/vnd.apache.avro;version=1.11.0"
        schema:
          $ref: "avro/order/OrderPlacedPayload.avsc"
      bindings:
        kafka:
          schemaIdLocation: header
          key:
            type: string
            description: Order ID used as partition key.
```

## Avro Schema Conventions

### Payload Schema

```json
{
  "type": "record",
  "name": "OrderPlacedPayload",
  "doc": "Published when a new order is successfully placed.",
  "fields": [
    {
      "name": "order_id",
      "type": "string",
      "doc": "Prefixed order identifier (ord_*)."
    },
    {
      "name": "customer_id",
      "type": "string",
      "doc": "Prefixed customer identifier (cus_*)."
    },
    {
      "name": "items",
      "type": {
        "type": "array",
        "items": "OrderItemData"
      },
      "doc": "Line items in the order."
    },
    {
      "name": "total_amount",
      "type": "long",
      "doc": "Total in smallest currency unit (e.g. cents)."
    },
    {
      "name": "currency",
      "type": "string",
      "doc": "Three-letter ISO 4217 currency code, lowercase."
    },
    {
      "name": "occurred_at",
      "type": "long",
      "logicalType": "timestamp-millis",
      "doc": "Timestamp when the event occurred."
    }
  ]
}
```

### Shared Types

```json
{
  "type": "record",
  "name": "OrderItemData",
  "doc": "A single line item in an order.",
  "fields": [
    {"name": "product_id", "type": "string", "doc": "Product identifier."},
    {"name": "quantity", "type": "int", "doc": "Number of units."},
    {"name": "unit_price", "type": "long", "doc": "Price per unit in smallest currency unit."}
  ]
}
```

### Naming Conventions

- **Record name**: `{EventName}Payload` — `OrderPlacedPayload`, `PaymentProcessedPayload`
- **Field names**: `snake_case` — consistent with OpenAPI and Kafka ecosystem conventions
- **Nested records**: extracted into separate `.avsc` files, referenced by full namespace

### Field Conventions

- Use prefixed IDs (`ord_`, `cus_`, etc.) — same as OpenAPI, type is `string`
- Monetary amounts as `long` in smallest currency unit + `currency` field
- Timestamps use `long` with `logicalType: timestamp-millis`
- Every field has a `doc` string
- Use `null` union for optional fields: `["null", "string"]` with `"default": null`

## Schema Evolution Rules

Avro + Schema Registry enforce compatibility. Follow these rules to maintain **backward compatibility** (new reader, old writer):

- **Add fields** with a `default` value — always safe
- **Remove fields** that have a `default` — safe (old readers ignore unknown fields)
- **Never remove** a field without a default
- **Never rename** fields — add a new field, deprecate the old one
- **Never change** a field's type (except widening: `int` → `long`, `float` → `double`)
- **Never reorder** fields in a way that changes semantics

## Channel Naming

- Dot-separated, lowercase: `order.events.placed`, `payment.events.processed`
- Pattern: `{aggregate}.events.{past-tense-verb}`
- One event type per channel (topic) for clean consumer routing
- Topic name matches channel address

## Kafka Bindings

- Aggregate ID as message key (partition ordering per aggregate)
- Schema ID in message header (`schemaIdLocation: header`)
- Correlation ID propagated via Kafka headers (not in the Avro payload)

## Event Headers

Kafka headers (not Avro-encoded) carry cross-cutting metadata:

| Header           | Type   | Description                              |
|------------------|--------|------------------------------------------|
| `correlationId`  | string | Correlation ID for distributed tracing   |
| `eventId`        | string | Unique identifier for this event instance|
| `eventType`      | string | e.g. `order.placed`                      |

These stay as Kafka headers (not in the Avro payload) because they're infrastructure concerns, not domain data.

## Schema Independence

Avro schemas are **independent** from OpenAPI schemas. They describe the same domain concepts but are maintained separately. AsyncAPI references the `.avsc` files but shares nothing with the OpenAPI spec.

Rationale: sync and async contracts evolve at different rates and serve different consumers.

## Roles Summary

| Artifact          | Role                                          |
|-------------------|-----------------------------------------------|
| `asyncapi.yaml`   | Contract documentation (channels, operations, bindings) |
| `.avsc` files      | Source of truth for payload schemas, drive codegen |
| Schema Registry    | Runtime compatibility enforcement              |

## Rules

- AsyncAPI 3.0 only, YAML format
- `.avsc` files are the source of truth for event payloads — AsyncAPI references them
- AsyncAPI documents channels, operations, and bindings — not used for codegen
- One event type per channel
- Channel address pattern: `{aggregate}.events.{past-tense-verb}`
- Record names: `{EventName}Payload`
- Field names in `snake_case`
- Every field has a `doc` string
- Use prefixed IDs consistent with OpenAPI (`ord_`, `cus_`, etc.)
- Monetary amounts as `long` in smallest currency unit + `currency`
- Timestamps use `long` with `logicalType: timestamp-millis`
- Optional fields use `["null", "type"]` with `"default": null`
- New fields must always have a `default` (backward compatibility)
- Never remove fields without defaults, never rename, never change types
- Cross-cutting metadata (correlation ID, event ID) goes in Kafka headers, not Avro payload
- Schemas are independent from OpenAPI — no cross-file references

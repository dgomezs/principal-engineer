# Contract-First API Development

Specs are the **single source of truth**. Code is generated from them. Never hand-edit generated code.

## Workflow

### Sync API (OpenAPI)

1. **Design** — write or update `openapi.yaml`
2. **Review** — spec changes are reviewed in the PR before any implementation
3. **Generate** — run codegen to produce server interfaces and DTOs from the spec
4. **Implement** — hand-write resource classes that implement the generated interfaces
5. **Validate** — tests verify the implementation against the contract

### Async API (AsyncAPI + Avro)

1. **Design** — write or update `.avsc` schemas and reference them from `asyncapi.yaml`
2. **Review** — schema changes are reviewed for backward compatibility before merging
3. **Generate** — run codegen to produce typed payload classes from `.avsc` files
4. **Register** — Schema Registry validates compatibility on deploy
5. **Implement** — hand-write consumers/producers that use the generated payload types
6. **Validate** — tests verify the implementation against the contract

## Sources of Truth

| Concern            | Source of truth          | Role                                    |
|--------------------|--------------------------|-----------------------------------------|
| Sync API contract  | `openapi.yaml`           | Drives REST interface + DTO codegen     |
| Async API contract | `asyncapi.yaml`          | Documents channels, operations, bindings|
| Event payloads     | `.avsc` files            | Drives Avro `SpecificRecord` codegen    |
| Schema evolution   | Schema Registry          | Enforces compatibility at runtime       |

## File Layout

```
apps/<service>/src/main/resources/api/
├── openapi.yaml              # Sync API contract (OpenAPI 3.1)
├── asyncapi.yaml             # Async API contract (AsyncAPI 3.0) — references .avsc files
└── avro/                     # Avro schemas (source of truth for event payloads)
    ├── common/               # Shared types across aggregates
    │   ├── EventHeaders.avsc
    │   └── Money.avsc
    ├── order/                # Per-aggregate schemas
    │   ├── OrderPlacedPayload.avsc
    │   ├── OrderConfirmedPayload.avsc
    │   ├── OrderCancelledPayload.avsc
    │   └── OrderItemData.avsc
    └── payment/
        └── PaymentProcessedPayload.avsc
```

## Generated Code

- Generated output lives outside the source tree — never committed to git
- Developers run codegen once after clone for IDE autocompletion
- Dev mode / watch mode runs codegen automatically where available

### Generated Output (illustrative)

```
generated/
├── openapi/          # From openapi.yaml — server interfaces + DTOs
└── avro/             # From .avsc files — typed payload classes
```

## What Gets Generated vs Hand-Written

| Generated (never edit)                        | Hand-written                                |
|-----------------------------------------------|---------------------------------------------|
| REST resource interfaces (from OpenAPI)       | Resource classes implementing the interface  |
| Request/response DTOs (from OpenAPI)          | Mappers (generated DTO ↔ domain)            |
| Avro `SpecificRecord` classes (from `.avsc`)  | Kafka consumers and producers                |
|                                               | Mappers (Avro SpecificRecord ↔ domain)       |
|                                               | Command/query handlers                       |
|                                               | Domain entities, value objects, ports         |

## Schema Independence

OpenAPI schemas and Avro schemas are **independent**. They describe the same domain concepts but are maintained separately — no shared references across spec files.

Rationale: sync and async contracts evolve at different rates and serve different consumers.

## Rules

- Specs are the single source of truth — generated code is disposable
- Never hand-edit generated files
- Generated code is not committed to git
- Spec changes require PR review before implementation begins
- Avro schema changes must be reviewed for backward compatibility
- Regenerate after every spec change
- Every public API field in OpenAPI must have a `description`
- Every Avro field must have a `doc` string
- OpenAPI and Avro schemas are independent — no cross-references

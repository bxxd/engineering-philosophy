# Architecture

Languages, separation of concerns, and hexagonal structure. The skeleton every project hangs on.

## Languages

Rust first. TypeScript second. Python only if necessary (Poetry).

Each project picks a default stack and commits to it. List the stack once, in the project's `CLAUDE.md`, and don't deviate without evidence (benchmark, missing capability, hard requirement).

## Separation of concerns

Each concern lives in its own module, and each module is independently testable.

- Networking, persistence, domain logic, presentation — different concerns, different modules.
- If you can't test a piece in isolation, the seams are wrong.
- Concerns leak when modules know about each other's internals — guard the boundaries.
- The instinct that says "this code is doing too many things" is almost always right; trust it.

This flows directly into hexagonal architecture — the architectural expression of the same principle: name the seams (ports), put the infrastructure-aware code behind them (adapters), keep the core ignorant of the edges.

## Prefer hexagonal architecture

Domain core in the middle. Ports (traits) at the boundaries — DB, HTTP, message bus, external service. Adapters (Postgres, axum handler, Kafka client, S3 client) live at the edges, behind those ports.

- The core doesn't know how it's called or where data lives.
- Swap a Postgres adapter for an in-memory one in tests — the core doesn't notice.
- Domain types belong to the core, not the adapter.
- Each adapter independently testable; that falls out of the structure for free.

Tension with KISS: traits at the **architectural boundary** earn their keep — they isolate the core from infrastructure churn, make tests trivial, and let you replace a vendor without touching domain code. Traits *inside* the core (one impl, no real boundary) are still abstraction tax. Put traits where the seam is real; nowhere else.

```rust
// port — defined in the core, depends on nothing
pub trait OrderRepo {
    async fn find(&self, id: &str) -> Result<Order, RepoError>;
    async fn save(&self, order: &Order) -> Result<(), RepoError>;
}

// adapter — lives at the edge, depends on Postgres
pub struct PostgresOrderRepo { pool: PgPool }
impl OrderRepo for PostgresOrderRepo { /* ... */ }

// in-memory adapter for tests
pub struct InMemoryOrderRepo { store: Mutex<HashMap<String, Order>> }
impl OrderRepo for InMemoryOrderRepo { /* ... */ }

// core service — generic over the port, knows nothing about Postgres
pub struct OrderService<R: OrderRepo> { repo: R }
```

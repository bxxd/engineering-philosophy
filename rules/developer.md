# Engineering Philosophy

## Principles

### KISS — Keep It Simple, Stupid

Simple over clever in every decision.

- Favor the simplest solution that meets requirements.
- Implement the minimum viable solution first.
- Add complexity only when necessary and justified — by benchmarks, by a real need, not by imagination.
- Question every abstraction — is it really needed, or is it a trait waiting for a second implementation that never comes?
- Avoid over-engineering and premature optimization.
- If a junior dev can't read it in 30 seconds, it's too complex.

✅ Direct:
```rust
async fn get_user(id: &str) -> Result<User, ApiError> {
    db.query_one("SELECT * FROM users WHERE id = $1", &[id]).await
}
```

❌ Abstraction tax:
```rust
trait UserRepository {
    async fn find_by_id(&self, id: &str) -> Result<User, ApiError>;
}
struct PostgresUserRepository { /* ... */ }
impl UserRepository for PostgresUserRepository { /* ... */ }
```

### Don't change shit that works

Working code that looks ugly beats broken code that looks clean.

- No refactoring for style or aesthetics.
- If it works and meets requirements, leave it alone.
- The cost of "cleaning up" is usually paid in subtle regressions you didn't anticipate.

### One True Path (YOLO)

Single execution path. Pick the right approach and commit.

- No fallback logic, no defensive retries, no "just in case" alternative code paths.
- If the primary approach fails, that's valuable design feedback — not a reason to bolt on a backup.
- Avoid masking errors with workarounds. An error that surfaces is debuggable; an error that's silently caught is a time bomb.
- Clean, single-purpose code paths.
- Lock-in is intentional. Commit to your tools.

**Two paths = bug.** A second path masks errors, introduces drift (recalculation diverging from the original computation), and hides root causes. Pick the hot path, document it, ship.

✅ Clear:
```rust
async fn fetch_invoice(id: &str) -> Result<Invoice, ApiError> {
    let invoice = db.fetch_invoice(id).await?;
    Ok(invoice)
}
```

❌ Three paths, one bug:
```rust
async fn fetch_invoice(id: &str) -> Result<Invoice, ApiError> {
    if let Ok(inv) = db.fetch_invoice(id).await { return Ok(inv); }
    if let Ok(inv) = cache.get(id).await { return Ok(inv); }
    Ok(Invoice::default())  // silently hides DB and cache failures both
}
```

### DRY — One Source of Truth

Single source of truth for all logic and configuration.

- Eliminate code duplication through abstraction — but only after you see real repetition, not when you imagine it.
- Extract repeated patterns into reusable functions.
- Shared configuration in one place.
- Consistent imports and module structure across the workspace.
- Refactor when you see repetition (rule of three is a fine heuristic).

✅ Workspace dependencies, declared once:
```toml
[workspace.dependencies]
tokio = { version = "1.47", features = ["full"] }

[dependencies]
tokio = { workspace = true }
```

✅ Extract a common pattern that actually repeats:
```rust
async fn handle_record_request<F, T>(
    id: &str,
    operation: F,
) -> Result<Json<T>, ApiError>
where
    F: FnOnce(&Record) -> Result<T, ApiError>,
{
    let record = fetch_record(id).await?;
    let result = operation(&record)?;
    Ok(Json(result))
}
```

### Separation of concerns

Each concern lives in its own module, and each module is independently testable.

- Networking, persistence, domain logic, presentation — different concerns, different modules.
- If you can't test a piece in isolation, the seams are wrong.
- Concerns leak when modules know about each other's internals — guard the boundaries.
- The instinct that says "this code is doing too many things" is almost always right; trust it.

This flows directly into hexagonal architecture — the architectural expression of the same principle: name the seams (ports), put the infrastructure-aware code behind them (adapters), keep the core ignorant of the edges.

### Prefer hexagonal architecture

Domain core in the middle. Ports (traits) at the boundaries — DB, HTTP, message bus, external service. Adapters (Postgres, axum handler, Kafka client, S3 client) live at the edges, behind those ports.

- The core doesn't know how it's called or where data lives.
- Swap a Postgres adapter for an in-memory one in tests — the core doesn't notice.
- Domain types belong to the core, not the adapter.
- Each adapter independently testable; that falls out of the structure for free.

This is in *apparent* tension with the KISS example above. Reconciliation: traits at the **architectural boundary** earn their keep — they isolate the core from infrastructure churn, make tests trivial, and let you replace a vendor without touching domain code. Traits *inside* the core (one impl, no real boundary) are still abstraction tax. Put traits where the seam is real; nowhere else.

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

### Evidence-based optimization

Let data guide all decisions.

- Benchmark before optimizing.
- Use CLI tools to measure impact — `hyperfine`, `criterion`, `wrk`, `pprof`, `flamegraph`. Real numbers, not vibes.
- Prefer profiling over guessing. The bottleneck is rarely where you think it is.
- Profile to find actual bottlenecks, not assumed ones.
- Test assumptions with real data — real shapes, real volumes, real distributions.
- Use monitoring to measure the impact of changes in production.
- Document performance expectations in comments — they're contracts the next change has to honor.

✅ Document performance expectations:
```rust
/// Search articles with pagination.
///
/// Performance: Target <5ms p95 for 100 results.
/// Uses index on (author_id, published_at DESC).
async fn search_articles(author_id: &str, limit: u32) -> Result<Vec<Article>, ApiError> {
    // ...
}
```

## Operational

- `./tmp/` (not `/tmp/`) for scratch.
- No secrets in code.
- Never broadly `kill` / `pkill` by binary name — multiple projects can share the same path (e.g. `target/release/server`). Use `make kill` in the project dir, or kill by exact PID.
- Ask before touching prod.

## Summary

These principles work together to produce:

- **Simple code** that's easy to read, review, and maintain.
- **Focused execution paths** that fail fast and clearly, with no silent fallbacks.
- **Architectures with real seams** — hex at the boundaries, direct code in the core.
- **Systematic debugging** based on evidence, not guesswork.
- **No duplication** of logic, config, or dependencies.
- **Fast code** based on real measurements, not assumptions.
- **Clean signal** from compilers and linters, with warnings treated as bugs.

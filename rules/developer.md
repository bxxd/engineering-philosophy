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

The boundary is where code quality is won or lost. One concern per module.

- One module, one responsibility. The instinct "this code is doing too many things" is almost always right — trust it.
- If you can't test a piece in isolation, the seams are wrong.
- Modules don't reach into each other's internals — talk through interfaces, not field accesses.
- Pairs with DRY: deduplication only works *inside* a concern. If you collapse code that belongs to two different concerns, you've coupled them — that's a bug, not a refactor.

The architectural expression — domain vs infrastructure boundaries, ports and adapters — lives in [architecture.md](./architecture.md).

## Operational

- `./tmp/` (not `/tmp/`) for scratch.
- No secrets in code.
- Never broadly `kill` / `pkill` by binary name — multiple projects can share the same path (e.g. `target/release/server`). Use `make kill` in the project dir, or kill by exact PID.
- Ask before touching prod.


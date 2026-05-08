# Debugging

How to investigate, isolate, and fix bugs — systematically, evidence-first.

## Zero Effect Law — investigate before theorizing

The instinct is to form a theory and hunt for confirming evidence. Reverse it.

- Look at the data first.
- Find anything that might be relevant.
- *Then* form the theory.
- Most "obvious" causes are the second thing you find, not the first.

## Detective Mode for debugging

You have a theory of the crime; you need evidence before convicting. Treat debugging like a criminal investigation — systematic, evidence-based, methodical.

**1. Theory of the Crime — form a hypothesis.**
- What's the suspected root cause based on symptoms?
- What changed recently that could cause this?
- Analyze error messages for clues.
- Consider multiple suspects across layers and components.

**2. Gather Evidence — investigate the scene.**
- Add strategic logging at the crime scene (key execution points).
- Use `tracing::debug!()` to trace the timeline of events.
- Check database queries with `EXPLAIN ANALYZE` (forensics).
- Monitor system resources — CPU, memory, connections, file descriptors.
- Don't assume — collect facts.
- Document what you *find*, not what you *expect to find*.

**3. Divide and Conquer — isolate suspects.**
- Narrow down to a specific layer (HTTP, DB, external service, serialization).
- Test components independently to eliminate suspects.
- Use a binary search approach to isolate the problem area.
- Reproduce in a minimal test case — crime scene reconstruction.

**4. Prove the Theory — build the case.**
- Let facts on the ground prove the theory.
- Use reproducible test cases as evidence.
- Verify assumptions with benchmarks.
- Can you recreate the crime reliably? If not, you don't have a case.
- Document findings in code comments where they'll be seen again.

**5. Conviction — make the fix.**
- Make targeted fixes based on proven evidence.
- No guesswork. No shotgun debugging.
- Test that the fix addresses the *proven* root cause, not just the symptom.
- Clean up debugging code after resolution (or gate it behind a log level).
- Don't convict without evidence.

✅ Strategic debugging with evidence gathering:
```rust
async fn process_order(id: &str) -> Result<(), ApiError> {
    tracing::debug!("Starting order processing for id={}", id);

    let start = std::time::Instant::now();
    let order = db.fetch_order(id).await?;
    tracing::debug!("DB fetch took {:?}", start.elapsed());

    let start = std::time::Instant::now();
    let result = payment_service.charge(&order).await?;
    tracing::debug!("Payment service took {:?}", start.elapsed());

    Ok(result)
}
```

## If it's unusually slow, it's probably a bug

Don't accept "normal" slowness — investigate. Things can be fast. Code can be fast.

- **Sub-millisecond is achievable.** Aim for it. Sub-1ms response times in hot paths are not a fantasy — they're a target you should be looking for and surprised when you don't hit.
- Real example: 30ms → 0.6ms once we actually looked.
- Slow code is usually slow for a reason — N+1 queries, unbounded loops, sync I/O on an async path, allocations in a hot path, missing indexes, redundant serialization, lock contention.
- Debug systematically: measure, isolate, fix.
- "It's just slow" is a hypothesis, not a conclusion. Treat it like any other bug — gather evidence, find the cause, fix it.

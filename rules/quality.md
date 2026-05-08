# Code Quality Requirements

## Zero Warnings Policy

**CRITICAL: All code MUST compile with ZERO warnings. This is non-negotiable.**

Warnings indicate potential issues and clutter the build output, hiding real problems. They are often the visible tip of a deeper bug:

- Unused `Result` = an error you forgot to handle.
- Unused variable = abandoned logic, or a typo on the variable that *did* get used.
- `unwrap` / `expect` flagged by clippy = a panic surface waiting for prod traffic.
- Implicit `any` in TS = a type lie that will cost you later.
- Deprecated API = a known break in your future.
- Unreachable code = a control flow bug.
- Shadowed binding = ambiguity that will trip a future reader.

**Rules:**

- NEVER commit code with warnings.
- NEVER ignore warnings by letting them accumulate.
- CI will fail on any warnings:
  - `cargo clippy -- -D warnings`
  - `eslint --max-warnings=0`
  - `ruff check` / `mypy --strict`
- All checks must pass before merging.

If you must suppress one with `#[allow(...)]` or `// eslint-disable`, write the reason on the same line. "Came back clean" should mean *clean*, not "compiled."

## Go native

Use the platform's idiomatic patterns.

- Flag non-native approaches before implementing — there's almost always a reason the platform did it that way.
- Idiomatic code is reviewable, debuggable by anyone in the ecosystem, and won't surprise the next person.
- "I know better than the language designers" is a hypothesis you should be very slow to confirm.
- Native patterns get optimized by compiler authors and reviewed by the whole community. Custom patterns get optimized by you, alone, between other tasks.

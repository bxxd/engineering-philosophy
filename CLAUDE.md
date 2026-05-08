@rules/developer.md
@rules/architecture.md
@rules/debugging.md
@rules/quality.md
@rules/management.md

# Contributing to this repo

The rule files in `rules/` are drop-in. People install one file, two files, or all five into `.claude/rules/` — sometimes via curl, sometimes via "ask the agent." Each file must stand on its own.

## House rules for editing `rules/*.md`

- **No cross-links between rule files.** Don't write `[architecture.md](./architecture.md)` or "see `quality.md`" from inside another rule. Broken links read like a bug to anyone who installed a subset.
- **No assumptions about siblings.** Don't say "as discussed in X" or "the other file covers Y." Say what you mean inline.
- **Each file is readable by a complete newcomer.** No prerequisite reading from elsewhere in this repo.
- **Overlap is fine.** If a concept genuinely spans two files (e.g. separation of concerns lives at module level in `developer.md` and at architectural level in `architecture.md`), write it twice — once in each file's voice. The cost of a little repetition is much lower than a broken drop-in.
- **Tone.** Blunt, declarative, opinionated. Real examples with concrete numbers when you have them. No hedging, no "consider this." Bullet-heavy under each principle. Code blocks in the project's stack of choice (Rust first).

## When you change anything in `rules/`

- Update the README's "The rules" list if you add or rename a file.
- Update the install loop in the README to include any new file.
- Update the `@rules/...` references at the top of this `CLAUDE.md`.
- Update the agent-prompt option in the README so it lists the same files.
- Don't introduce a cross-link to fix a redundancy. Either keep both copies, or pick one home.

# agentic-engineering-philosophy

*The accumulated, crystallized knowledge of twenty years of coding and architecture across the stack — from C to Python to massively-scaled distributed computing to front-end and brand development, generalized.*

Rules for AI coding agents.

## The rules

- [rules/developer.md](./rules/developer.md) — KISS, YOLO, DRY, evidence-based optimization, operational guardrails.
- [rules/architecture.md](./rules/architecture.md) — language preferences, separation of concerns, hexagonal architecture.
- [rules/debugging.md](./rules/debugging.md) — Zero Effect Law, Detective Mode, sub-1ms targets.
- [rules/quality.md](./rules/quality.md) — Zero Warnings Policy, Go native.
- [rules/management.md](./rules/management.md) — Conway's Law as architecture context. Org structure is upstream of the seams the agent should propose.

## Install

Pick one. Both produce the same effect — the agent sees the rules every session.

### Option 1 — drop into `.claude/rules/`

Claude Code auto-loads any markdown in `.claude/rules/`.

**Project scope** (this project only):

```bash
mkdir -p .claude/rules
for f in developer.md architecture.md debugging.md quality.md management.md; do
  curl -fsSL "https://raw.githubusercontent.com/bxxd/agentic-engineering-philosophy/main/rules/$f" \
    -o ".claude/rules/$f"
done
```

**User scope** (every project on your machine):

```bash
mkdir -p ~/.claude/rules
for f in developer.md architecture.md debugging.md quality.md management.md; do
  curl -fsSL "https://raw.githubusercontent.com/bxxd/agentic-engineering-philosophy/main/rules/$f" \
    -o "$HOME/.claude/rules/$f"
done
```

### Option 2 — ask the agent

Paste this into your Claude Code / Cursor / Codex / etc. session:

> Fetch the five rule files at `https://raw.githubusercontent.com/bxxd/agentic-engineering-philosophy/main/rules/` (`developer.md`, `architecture.md`, `debugging.md`, `quality.md`, `management.md`) and add them to this project. Either save them to `.claude/rules/`, or `@`-reference them from `CLAUDE.md` / `AGENTS.md` — pick whichever fits the existing setup.

## Where to run the agent

These rules feed AI coding agents. Don't run those agents on the root of your home machine.

- Use an Incus container, a VM, or a separate physical box — anywhere with a fresh user account and contained blast radius.
- For a home lab, Tailscale ties the boxes together cleanly — laptop, sandbox host, anything else — over a private mesh, no port forwarding required.
- Agents have shell, file, and network access. A wrong command on your real home user touches your dotfiles, your SSH keys, your browser sessions, and anything else that account can reach. A container is a small price.

## License

MIT.

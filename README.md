# engineering-philosophy

One source of truth for engineering principles, fed to AI coding agents.

→ **[rules/developer.md](./rules/developer.md)**

## Install

Pick one. Both produce the same effect — the agent sees the principles every session.

### Option 1 — drop into `.claude/rules/`

Claude Code auto-loads any markdown in `.claude/rules/`.

**Project scope** (this project only):

```bash
mkdir -p .claude/rules
curl -fsSL https://raw.githubusercontent.com/bxxd/engineering-philosophy/main/rules/developer.md \
  -o .claude/rules/developer.md
```

**User scope** (every project on your machine):

```bash
mkdir -p ~/.claude/rules
curl -fsSL https://raw.githubusercontent.com/bxxd/engineering-philosophy/main/rules/developer.md \
  -o ~/.claude/rules/developer.md
```

### Option 2 — ask the agent

Paste this into your Claude Code / Cursor / Codex / etc. session:

> Fetch `https://raw.githubusercontent.com/bxxd/engineering-philosophy/main/rules/developer.md` and add it to this project. Either save it as `.claude/rules/developer.md`, or `@`-reference it from `CLAUDE.md` / `AGENTS.md` — pick whichever fits the existing setup.

## License

MIT.

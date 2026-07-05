# claude-skills

A Claude Code plugin with reusable skills for architectural documentation and spec-driven development.

## Install

From the terminal:

```
claude plugin marketplace add mapsa/claude-skills
claude plugin install mapsa@mapsa
```

Or from within a Claude Code session:

```
/plugin marketplace add mapsa/claude-skills
/plugin install mapsa@mapsa
```

Restart Claude Code after installing so the skills load.

## Skills

| Skill | Description | Invoke with |
|-------|-------------|-------------|
| [blueprint](skills/blueprint/) | Generates a living architectural blueprint in `.claude/docs/` — comprehensive enough to rebuild the project from scratch | `/blueprint` |

## License

MIT

# skills

Personal [Claude Code Agent Skills](https://docs.anthropic.com/en/docs/claude-code/skills) — background knowledge and workflows that auto-trigger during Claude Code sessions.

## Structure

Each skill lives in its own subdirectory with a `SKILL.md` containing YAML frontmatter and instructions.

```
skills/
├── bitcoin-node/          # Bitcoin Core, Electrs, Alby Hub, Sparrow, mining
├── homelab-context/       # Network topology, TrueNAS services, Docker, OPNsense, SSH
└── research-methodology/  # 6-lens analytical framework (references ~/Git/research-skill-graph/)
```

All skills use `user-invocable: false` — Claude loads them automatically when relevant, no slash command needed.

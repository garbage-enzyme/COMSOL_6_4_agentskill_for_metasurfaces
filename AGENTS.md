# AGENTS.md — cross-tool entry point

This repository ships two agent skills:

```text
skills/comsol-64-metasurface/SKILL.md
skills/comsol-mcp-development/SKILL.md
```

Use `comsol-64-metasurface` for modeling, solver operation, host recovery, and
scientific evidence. Use `comsol-mcp-development` only for server code, tests,
CI, packaging, deployment, and review maintenance. Read the selected short
routing entry, then each routed reference completely. Do not preload every
reference module.

The progressive-disclosure layout is intentionally portable across Claude Code,
Codex CLI, and opencode:

- `SKILL.md` contains standard YAML `name` and `description` frontmatter plus the
  short core workflow.
- `references/*.md` contains task-specific detail linked with relative paths.
- No platform-specific tool protocol is required to follow the skill.

The operation modules cover clientapi, Acoustics/PDE, periodic Wave Optics,
durable solver use, host recovery, evidence, recipes, and troubleshooting. The
development modules cover public contracts, runtime reliability, test/CI
policy, release packaging, deployment identity, explicitly requested legacy
compatibility, and review maintenance.

Do not publish usernames, home directories, host-specific thresholds,
credentials, private result values, or internal project/phase labels in this
repository.

Companion MCP server:
https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated

# COMSOL 6.4+ agent skills

[中文](README_CN.md)

Two progressive-disclosure skills:

- `comsol-64-metasurface` for COMSOL modeling, solver operation, host recovery,
  and physical validation;
- `comsol-mcp-development` for server engineering, tests, CI, packaging,
  deployment, and review maintenance.

## CLI compatibility

The same skill folder works with Claude Code, Codex CLI, opencode, and Hermes
Agent.

| CLI | Entry |
| --- | --- |
| Claude Code | Copy the complete folder to `~/.claude/skills/comsol-64-metasurface/`; a project-local `CLAUDE.md` import also remains supported. |
| Codex CLI | Install the folder under the Codex skills directory or point to it from `AGENTS.md`. |
| opencode | Copy the folder under `~/.config/opencode/skills/`. |
| Hermes Agent | Install under `~/.hermes/skills/` or expose the repository `skills/` directory through `skills.external_dirs`. |

On Windows 11, Claude Code 2.1.220 was verified to discover a personal install
under `~/.claude/skills/comsol-64-metasurface/` after all 10 files were copied
byte-for-byte and their SHA-256 values matched the source. This establishes
installation and skill discovery only. The acceptance run did not start COMSOL
or execute the documented modeling, solve, or evidence workflows.

Hermes Agent compatibility was checked against its official
[Skills System documentation](https://github.com/NousResearch/hermes-agent/blob/main/website/docs/user-guide/features/skills.md).
This repository follows the Agent Skills layout Hermes scans:
a directory-named skill with `SKILL.md`, required `name` and `description`
frontmatter, and relative supporting files under `references/`. Hermes-specific
frontmatter is optional, so no Hermes-only fork is required. Loading the skill
does not install COMSOL or expose a solver: the Hermes session must still have a
working COMSOL MCP connection or terminal access to the documented MPh/clientapi
environment.

`SKILL.md` is intentionally short. Its routing table links task-specific files
under `references/`, so an agent loads only the modules needed for the current
task. All links are relative and no platform-specific tool protocol is required.

This structure is also a practical compatibility choice: some agent runtimes
enforce reading the complete `SKILL.md` on every matching turn. Keeping that
mandatory entry short avoids repeated latency and context cost, while the routed
reference modules retain the full operational knowledge and are read completely
when their task area is needed.

## Coverage

Operational skill:

- standalone `ModelClient` overload and geometry-probing traps;
- typed Pressure Acoustics, mathematical PDE, and named selections;
- periodic ports, incidence angles, polarization, CopyFace meshes, oblique cells;
- Drude/loss signs, layered boundaries, dispersive sweeps, PML/manual Floquet;
- durable jobs, cancellation, Windows sharing/identity stability, host recovery;
- R/T/A, physical flux closure, wavelength sync, provenance, convergence, fields;
- MIM, grating, nanopillar, parameter-scan, and field-export recipes.

Development skill:

- public tool/profile/schema/settings contracts and bounded validation;
- durable state, process ownership, cancellation, admission, and Windows I/O;
- solver-free tests, hosted CI policy, warning cleanup, and gate diagnosis;
- wheel/sdist boundaries, non-editable deployment, and installed build identity;
- hierarchical review validation, repair ledgers, exact-SHA CI, and release flow.

## Install

Clone the repository, then copy each complete skill folder you need—not only
its `SKILL.md`:

```text
skills/comsol-64-metasurface/
skills/comsol-mcp-development/
```

Example for opencode:

```bash
mkdir -p ~/.config/opencode/skills
cp -r skills/comsol-64-metasurface ~/.config/opencode/skills/
```

Example for a Codex personal install:

```bash
mkdir -p ~/.codex/skills
cp -r skills/comsol-64-metasurface ~/.codex/skills/
```

Example for a Claude Code personal install:

```powershell
New-Item -ItemType Directory -Force "$HOME\.claude\skills" | Out-Null
Copy-Item -Recurse -Force "skills\comsol-64-metasurface" "$HOME\.claude\skills\"
```

Example for Hermes Agent using its direct GitHub skill installer:

```bash
hermes skills install garbage-enzyme/COMSOL_6_4_agentskill_for_metasurfaces/skills/comsol-64-metasurface
```

Alternatively, copy the complete folder to `~/.hermes/skills/` or add the
repository's `skills/` directory to `skills.external_dirs` in
`~/.hermes/config.yaml`.

For Claude Code project use, keep `CLAUDE.md` and `skills/` together. For a
personal configuration, install the complete folder under `~/.claude/skills/`
and start a fresh session so discovery is not confused with a stale process.

## Layout

```text
skills/comsol-64-metasurface/
├── SKILL.md
└── references/
    ├── acoustics-pde.md
    ├── clientapi-core.md
    ├── durable-runtime.md
    ├── host-resource-recovery.md
    ├── magnetic-fields.md
    ├── materials-boundaries.md
    ├── mcp-offline.md
    ├── plotting.md
    ├── troubleshooting.md
    ├── validation-evidence.md
    ├── wave-optics-periodic.md
    └── workflow-recipes.md

skills/comsol-mcp-development/
├── SKILL.md
├── agents/openai.yaml
└── references/
    ├── packaging-release.md
    ├── repository-contracts.md
    ├── review-maintenance.md
    ├── runtime-reliability.md
    └── testing-ci.md
```

The published content is portable: it contains no usernames, home directories,
host-specific thresholds, credentials, private spectra, or internal project
phase names.

Companion server:
https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated

License: MIT.

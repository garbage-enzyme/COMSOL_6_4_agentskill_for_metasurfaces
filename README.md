# COMSOL 6.4+ agent skill

[中文](README_CN.md)

A progressive-disclosure skill for using COMSOL Multiphysics 6.4+ through a
COMSOL MCP server or MPh 1.3.1 standalone/clientapi, with periodic Wave Optics,
durable solver workflows, and physical validation.

## CLI compatibility

The same skill folder works with Claude Code, Codex CLI, opencode, and Hermes
Agent.

| CLI | Entry |
| --- | --- |
| Claude Code | Import `skills/comsol-64-metasurface/SKILL.md` from `CLAUDE.md`. |
| Codex CLI | Install the folder under the Codex skills directory or point to it from `AGENTS.md`. |
| opencode | Copy the folder under `~/.config/opencode/skills/`. |
| Hermes Agent | Install under `~/.hermes/skills/` or expose the repository `skills/` directory through `skills.external_dirs`. |

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

- standalone `ModelClient` overload and geometry-probing traps;
- periodic ports, incidence angles, polarization, CopyFace meshes, oblique cells;
- Drude/loss signs, layered boundaries, dispersive sweeps, PML/manual Floquet;
- durable jobs, cancellation, Windows sharing/identity stability, resource gates;
- R/T/A, physical flux closure, wavelength sync, provenance, convergence, fields;
- MIM, grating, nanopillar, parameter-scan, and field-export recipes.

## Install

Clone the repository, then copy the complete skill folder—not only `SKILL.md`:

```text
skills/comsol-64-metasurface/
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

Example for Hermes Agent using its direct GitHub skill installer:

```bash
hermes skills install garbage-enzyme/COMSOL_6_4_agentskill_for_metasurfaces/skills/comsol-64-metasurface
```

Alternatively, copy the complete folder to `~/.hermes/skills/` or add the
repository's `skills/` directory to `skills.external_dirs` in
`~/.hermes/config.yaml`.

For Claude Code project use, keep `CLAUDE.md` and `skills/` together. For a
personal Claude configuration, import the installed `SKILL.md` from the relevant
`CLAUDE.md`.

## Layout

```text
skills/comsol-64-metasurface/
├── SKILL.md
└── references/
    ├── clientapi-core.md
    ├── durable-runtime.md
    ├── materials-boundaries.md
    ├── troubleshooting.md
    ├── validation-evidence.md
    ├── wave-optics-periodic.md
    └── workflow-recipes.md
```

The published content is portable: it contains no usernames, home directories,
host-specific thresholds, credentials, private spectra, or internal project
phase names.

Companion server:
https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated

License: MIT.

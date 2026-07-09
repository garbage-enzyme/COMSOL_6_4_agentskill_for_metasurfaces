# AGENTS.md — Codex / cross-tool entry point

This repository ships an **agent skill** at:

```
skills/comsol-64-operations/SKILL.md
```

It is a focused instruction document (with YAML frontmatter) that teaches coding agents how to drive **COMSOL Multiphysics 6.4+** through the [comsol MCP server](https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated) with MPh 1.3.1 standalone / `clientapi`, and how to write or fix code in `src/tools/` when the clientapi wrapper's API differs from the direct `com.comsol.model.Model` API. Also covers geometry boundary probing (getUpDown/faceX/faceNormal), fin Form Assembly (imprint=True, createpairs=False), Wave Optics ewfd periodic metasurface setup with MIM Drude LayeredTransition BC.

## When to read it

Read `skills/comsol-64-operations/SKILL.md` **before** doing any of these:

- Driving COMSOL 6.4+ via the comsol MCP server (any `comsol_*` tool call).
- Writing or modifying code in the companion MCP server's `src/tools/` directory.
- Debugging errors like `No matching overloads`, `Operation_cannot_be_created_in_this_context`, `'ComponentGeomListClient' object is not subscriptable`, or `'PhysicsFeatureClient' object has no attribute 'type'` on 6.4 standalone.

The skill's frontmatter `description` is the matching signal for agents that support skill auto-loading (opencode, Claude Code). Codex reads AGENTS.md directly, so: **treat the skill file as required reading for COMSOL-related tasks.**

## Why this skill exists

Under `mph.Client(cores=...)` (MPh 1.3+ standalone), `model.java` returns `com.comsol.clientapi.impl.ModelClient` — a wrapper whose method overloads differ from the direct `com.comsol.model.Model` the upstream MCP code was written against. The skill documents every mismatch found + fixed in the companion fork, plus 6.3/6.4-specific traps (Electrostatics `fsp1` FreeSpace ignoring material ε_r, Block boundary numbering, `Terminal` V0 unreliable, `(1[V])^2` expression syntax, missing `m.study()`, `selection().entities()`, `geom.getUpDown()` no-arg form, fin FormAssembly via `action='assembly'+imprint=True`).

Verified end-to-end: parallel-plate capacitor **C = 1.8593794420 pF** vs theory 1.8593794407 pF (err 7 × 10⁻¹⁰ pF); MIM paper baseline (Chen et al. 2023) R profile matches Drude-continuous-Au prediction across 1–10 µm.

## Companion code

The calibrated MCP server source lives at: https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated

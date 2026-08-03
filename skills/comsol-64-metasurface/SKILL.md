---
name: comsol-64-metasurface
description: COMSOL Multiphysics 6.4+ and MPh 1.3.1 operations, startup settings, execution-mode selection, modeling, durable solves, evidence validation, and recovery through COMSOL MCP or standalone/clientapi. Use when configuring the shared settings or Settings GUI; choosing interactive, inline, launcher, standalone, or MPH-only execution; driving COMSOL through MCP or mph.Client; building Acoustics, PDE, Heat Transfer, Solid Mechanics, Wave Optics, thermal-emission, thermo-optomechanical, or metasurface models; collaborating with local Server/Desktop; debugging clientapi, selection, mesh, port, material, or study failures; recovering after resource exhaustion or MCP disconnection; running resumable sweeps; comparing spectral line shapes; or auditing polarization, passivity, power closure, wavelength synchronization, convergence, provenance, resource admission, and solver ownership.
---

# COMSOL 6.4+ operations

Use this file as the short entry point. Read only the reference modules required
for the current task. All paths below are relative so the same folder works in
Claude Code, Codex CLI, and opencode.

## Mandatory operating rules

1. Inspect live capabilities and solver ownership before any COMSOL action.
2. Keep one solver owner. If a lease or external COMSOL/MPh process exists, do
   not construct another client.
3. Treat source models as immutable. Apply mutations only to provenance-tracked
   derived copies and verify the source SHA-256 afterward.
4. Use one point per solve for fragile or long sweeps. Persist each validated row
   with flush and `fsync`; resume only exact configuration identities.
5. Never infer physical polarization from `S/P`, energy closure from shared
   internal normalizations, convergence from a fixed-wavelength amplitude, or
   solver progress from CPU/disk activity alone.
6. Require explicit caller policy for scientific classification and resource
   thresholds. Do not invent host defaults.
7. Stop before a visual mode claim unless an image-capable reviewer has received
   the exact bounded artifacts and returned a review receipt.
8. Keep responses, journals, queues, retries, subprocesses, and artifact sizes
   bounded. Fail closed when process identity, cleanup, telemetry, or rollback
   evidence is uncertain.
9. Keep research scope, publication standards, and project priorities
   caller-owned. Describe capabilities and evidence contracts rather than a
   default project strategy.
10. In a shared Desktop/Server session, require explicit local endpoint and
    model adoption, take turns rather than editing simultaneously, preserve the
    user's Server/Desktop/model on detach, and treat GUI visibility as distinct
    from verified scientific evidence.
11. Prefer the Settings GUI for ordinary users: call `settings.start` once,
    pause while the user owns the GUI, and never edit JSON while it is open.
    Reserve direct JSON editing for developers, automation, recovery, or an
    explicit user request. Keep one shared settings file for every agent and do
    not create agent-specific copies.
12. Serialize every call to one COMSOL MCP stdio server, including read-only
    discovery and status calls. Never use `Promise.all`, concurrent tool batches,
    or overlapping lifecycle polls against the same server.

## Strict MCP transport sequence

Treat tool-level concurrency metadata as server-side operation policy, not
permission to send parallel stdio requests. Call `capabilities`,
`comsol_status`, and `solver_status` one at a time. For a lifecycle gate, use
this exact order:

```text
capabilities -> comsol_status -> solver_status -> comsol_start once
-> serial comsol_status polls -> comsol_disconnect -> solver_status
-> comsol_start once -> serial comsol_status polls
-> comsol_disconnect -> solver_status
```

Do not retry a timed-out request by issuing another COMSOL MCP call while the
original request may still be executing. Parallelize only work that does not
call the same COMSOL MCP server and cannot overlap its solver lifecycle.

## Shared settings contract

Use the installed Settings GUI as the normal user configuration path. It writes
the same shared settings file used by every agent. Treat the checked-in
`settings.json` template as an advanced interface for developers, automation,
recovery, and explicitly authorized agent edits. Missing entries use safe
defaults. An illegal value falls back only that entry and is reported through
`capabilities` or `evidence_integrity_status` as a bounded `settings_errors`
item; malformed JSON falls back to the complete safe default document and
reports the error. Check `project_settings.configuration_state` before relying
on a profile or path.

For shared Desktop/Server work, choose any valid base profile and enable the
independent shared-server feature in the GUI Profile tab. The advanced JSON
equivalent is `shared_server.enabled=true`; no `desktop_shared` profile exists.
Restart the MCP host after startup-setting changes. Evidence-integrity checks
remain default-on; only an explicit GUI opt-out or JSON boolean `false` may
disable a check, and it must propagate `strictly_verified: false`. Treat the old
individual environment variables as compatibility overrides, not the normal
multi-agent configuration.

## Simulation execution modes

Choose the execution mode before starting a run. Mode selection is agent
workflow guidance, not an MCP operation; do not look for or invent a generic
mode-recommendation tool.

- Use `interactive` for incremental MCP edits, readback, and short feedback.
  Ordinary interactive work is distinct from the default-off shared
  Desktop/Server collaboration path, which requires explicit user intent.
- Use `inline` for one bounded Python script, dry run, smoke test, or short
  simulation conservatively estimated below one hour. This boundary is a
  planning heuristic, not a timeout or success guarantee. Inline has no
  automatic resume promise.
- Use `launcher` for long local work, multi-point campaigns, unattended
  monitoring, or point-boundary pause/resume when local Python is available.
  The driver must flush and `fsync` each completed point before continuing.
- Use `standalone` only for a cross-device Windows 10/11 x64 target with
  installed and licensed COMSOL 6.4 that needs launcher-like behavior without
  target Python. It still requires COMSOL and its bundled Java runtime.
- Use `mphonly` when the required handoff is one final portable MPH, commonly
  for a COMSOL-managed cluster or cloud environment. A COMSOL Job Configuration
  checkpoint can recover only to its latest checkpoint; do not claim exact
  per-point durability. One final MPH does not mean zero temporary, log,
  synchronization, or recovery files during execution.

Default to `interactive`, `inline`, or `launcher`. Before preparing
`standalone` or `mphonly`, ask for the target operating system and architecture,
exact COMSOL build/modules, license arrangement, Python and Java constraints,
scheduler/submission interface, storage and path rules, network access, resource
limits, and required status/resume/output behavior. Missing target facts require
a clarification request, not a guessed package.

## Reference router

Read each selected file completely before acting.

| Task | Read |
| --- | --- |
| `ModelClient` overloads, components, geometry probing, electrostatics, heat transfer, study/result basics | [clientapi-core.md](references/clientapi-core.md) |
| Named Box/side selections, Pressure Acoustics, Coefficient/General/Weak Form PDE, atomic boundary batches, analytical duct/Poisson validation | [acoustics-pde.md](references/acoustics-pde.md) |
| AC/DC magnetic-field interfaces, Coil features, Java-tag conversion, standalone cleanup, and one-point smoke | [magnetic-fields.md](references/magnetic-fields.md) |
| **Only when the required COMSOL operation is unavailable through MCP**: direct `mph` installation and standalone fallback, with manual ownership and evidence guards | [mcp-offline.md](references/mcp-offline.md) |
| `PeriodicStructure`, `rdir1`, incidence angles, polarization, periodic mesh, oblique cells | [wave-optics-periodic.md](references/wave-optics-periodic.md) |
| Drude/loss signs, layered boundaries, dispersive sweeps, PML, manual Floquet | [materials-boundaries.md](references/materials-boundaries.md) |
| Solver ownership, shared Desktop/attached Server, durable jobs/validation matrices, cancellation, Windows load stability, resource telemetry/admission | [durable-runtime.md](references/durable-runtime.md) |
| Heavy-solve resource exhaustion, vanished MCP transport, durable-progress preservation, safe pause/recovery, and exact resume | [host-resource-recovery.md](references/host-resource-recovery.md) |
| Settings GUI onboarding, `settings.start` handoff, editing ownership, legacy metadata, and post-save restart | [gui.md](references/gui.md) |
| Default-on evidence integrity, R/T/A, flux closure, polarization evidence, wavelength sync, provenance, convergence, fields | [validation-evidence.md](references/validation-evidence.md) |
| Solver-free configuration comparison, Lorentzian/Fano line-shape comparison, thermal material/radiation evidence, and thermal-to-optical replay | [thermal-spectral-workflows.md](references/thermal-spectral-workflows.md) |
| MIM, gratings, nanopillars, parameter scans, field export, common modeling recipes | [workflow-recipes.md](references/workflow-recipes.md) |
| Native result plots, boundary-selected surfaces, shared color ranges, camera/view handling, and PNG export | [plotting.md](references/plotting.md) |
| Error signatures and the smallest safe diagnostic | [troubleshooting.md](references/troubleshooting.md) |

For a task spanning several areas, read only their union. Examples:

- Periodic angle sweep: periodic + durable runtime + validation evidence.
- New dispersive metasurface model: clientapi + periodic + materials + evidence.

## Default execution sequence

1. Discover the active tool/profile surface; treat live discovery as authority.
2. Query ownership/status without starting COMSOL.
3. Hash the source and normalize the exact requested configuration.
4. Run solver-free preflight first: topology, selections, expressions, policy,
   artifact paths, resource availability, and immutable identity.
5. Acquire ownership once; then create/load the client and derived model.
6. Run the smallest diagnostic or one-point gate before a sweep.
7. Persist raw values and assumptions before interpretation.
8. Release model/client/worker/descendants/port/lease and verify absence.
9. Re-hash the source and archive a path-redacted evidence manifest.

## Acceptance language

Use `verified`, `measured`, `derived_from_declared_convention`, `label_only`,
`unknown`, `not_requested`, and `not_applicable` precisely. A successful native
call is not by itself physical validation. A requested cancellation is not a
terminal cancellation. A read-only mesh audit is not node-equality proof. An
all-air result proves port/mesh consistency, not angle mapping or target
polarization in the physical structure.

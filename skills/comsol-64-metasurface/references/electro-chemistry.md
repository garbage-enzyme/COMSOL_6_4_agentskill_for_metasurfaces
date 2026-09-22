# Electrochemistry Module profile

Use this reference only when the MCP host is explicitly configured with the
`electro_chemistry` profile. The profile is experimental and isolated: its
tools are not part of `core`, `full`, `experimental`, or
`comsolless_read_only`. Restart the MCP host after changing the profile, then
verify the live catalog and tool names through `capabilities`.

## Public surface

The current profile exposes five bounded tools:

- `electro_chemistry_catalog` — solver-free declared interfaces, feature types,
  required products, and resource policy. It does not probe a license or claim
  that a Java object exists.
- `electro_chemistry_inspect` — read-only tag, label, type, and selection
  inspection for one existing physics interface.
- `physics_add_electrochemistry` — creates one supported electrochemistry
  physics interface with duplicate-tag refusal and rollback on failure.
- `physics_configure_electrode_reaction` — creates an `ElectrodeSurface`
  boundary. On the accepted COMSOL 6.4.0.293 host, the verified property is
  surface resistivity `rhos`; Butler-Volmer/Tafel parameters are rejected when
  the selected interface does not expose them.
- `physics_set_electrolyte` — creates an `Electrolyte` feature on selected
  domains and optionally sets ionic conductivity `sigmal`.

Every mutation remains caller-owned and must preserve source-model immutability,
PathPolicy, bounded selections, rollback, cleanup, and evidence separation.
Profile selection does not choose a solver, mesh, core count, out-of-core mode,
or scientific resource budget.

## COMSOL 6.4 ClientAPI facts

For the verified MPh 1.3.1 / COMSOL 6.4.0.293 clientapi path:

```python
component.physics().create(
    "sec", "SecondaryCurrentDistribution", "3"
)
electrode = physics.feature().create("es1", "ElectrodeSurface", 2)
electrode.selection().set([boundary_id])
electrode.set("rhos", "1")
electrolyte = physics.feature().create("eip1", "Electrolyte", 3)
electrolyte.selection().set([domain_id])
electrolyte.set("sigmal", "1")
```

Physics creation uses a string spatial dimension (`"3"`); feature creation uses
an integer entity dimension (`2` for boundaries, `3` for domains). Convert Java
tag arrays with `list(...)` and retrieve objects by tag with `.get(tag)`.

The minimal live probe verified `SecondaryCurrentDistribution`, its default
features, `ElectrodeSurface.rhos`, `Electrolyte`, `ElectrolytePotential`, and
`ElectrodePotential`. `TertiaryCurrentDistribution` and `Electroanalysis` were
rejected as unknown interface names on that exact COMSOL build. Do not broaden
the catalog from an upstream COMSOL 6.3 record; probe the exact installed build
before adding an interface or property.

## Evidence and limits

The profile catalog is a declaration, not scientific validation. A successful
native creation proves API availability and cleanup only; it does not prove a
valid electrochemical model, material law, boundary condition, mesh, study, or
scientific result. Use a fresh solver-owner preflight, one serial live probe,
and a redacted receipt containing runtime identity, module findings, rollback,
lease release, and process cleanup. Never run a scientific solve as part of the
profile smoke test unless the caller separately supplies a scientific policy and
resource budget.

If an interface or feature is unavailable, return the explicit module/product
error and preserve the model unchanged. Do not silently fall back to
Electrostatics or a generic physics interface.

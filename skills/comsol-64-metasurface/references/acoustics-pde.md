# Acoustics and mathematical PDE

## Contents

- Profile and tool discovery
- Named selections
- Pressure Acoustics
- Mathematical PDE interfaces
- Boundary contracts
- Minimal physical validation
- Save and cleanup order

## Profile and tool discovery

Use `basic_fem` or `full`, restart the MCP host after changing the profile, and
treat live discovery as authority. The typed surface includes:

```text
geometry_create_box_selection
geometry_create_side_selections
physics_get_acoustic_boundary_conditions
physics_get_pde_boundary_conditions
physics_add_pressure_acoustics
physics_add_coefficient_form_pde
physics_add_general_form_pde
physics_add_weak_form_pde
physics_configure_acoustic_boundary
physics_setup_acoustic_boundaries
physics_configure_pde_boundary
physics_setup_pde_boundaries
```

These tools use bounded exact tags, expression sizes, property allowlists, and
transactional rollback. Do not replace them with arbitrary physics-type or
property pass-through.

## Named selections

Create selections only after `geometry.run()` and inspect their resolved entity
IDs. A 2D rectangular four-side request creates `<prefix>_left`, `_right`,
`_bottom`, and `_top` atomically. It must return one distinct boundary per side
before those names are used for physics.

Use Box `condition="inside"` for thin side boxes. `intersects` also includes
boundaries that touch the box at a corner and selected three boundaries per side
in a COMSOL 6.4 rectangle probe. Keep a small declared tolerance around the side
plane; do not infer stable entity numbers from construction order.

## Pressure Acoustics

The exact clientapi creation is:

```python
acoustics = component.physics().create("acpr", "PressureAcoustics", "geom1")
```

The default fluid feature tag on COMSOL 6.4.0.293 is `fpam1`. User-defined air
properties can be set with `rho_mat="userdef"`, `rho`, `c_mat="userdef"`, and
`c`. Verify these tags and properties on a new COMSOL release family.

The verified typed boundary set is:

```text
SoundHard
SoundSoft
Pressure               p0
Impedance              Zn
NormalAcceleration     nacc
NormalVelocity         nvel
PlaneWaveRadiation
```

`SphericalWaveRadiation` is not in the public contract because it did not create
on the validated 2D interface. Do not infer that a dimension-specific COMSOL
feature is portable through the generic boundary tool.

## Mathematical PDE interfaces

Create the exact interfaces with a Java `String[]` of unique dependent
variables:

| Form | Interface type | Default equation tag | Equation properties |
| --- | --- | --- | --- |
| Coefficient | `CoefficientFormPDE` | `cfeq1` | `c`, `a`, `f`, `da`, `ea`, `al`, `be`, `ga` |
| General | `GeneralFormPDE` | `gfeq1` | `Ga`, `f`, `da`, `ea` |
| Weak | `WeakFormPDE` | `wfeq1` | `weak` |

Dependent-variable names must be unique across every active interface in one
component. Reusing `u` for Coefficient, General, and Weak Form in one component
causes COMSOL to reject duplicate variable names. A General Form `Ga` is a
spatial flux vector; supply the shape required by the model dimension instead
of assuming one scalar string is valid.

Do not include incomplete General or Weak interfaces in the study used to solve
an unrelated Coefficient Form model. Their unconstrained degrees of freedom can
make the stationary system singular. Solve the complete intended interface set
or create no-solve interface probes after the validated solve.

## Boundary contracts

The verified PDE boundary set is:

```text
DirichletBoundary      r
FluxBoundary           g, q
ZeroFluxBoundary
PeriodicCondition
```

`WeakContribution` is excluded from the typed boundary helper. Its weak
expression uses an indexed clientapi overload, which the generic JSON property
transport does not express safely. Configure indexed weak contributions only
through a separately typed and release-validated operation.

Supply exactly one of numeric boundaries or one existing named selection. Batch
configuration is atomic: if any create, selection, property, or label operation
fails, remove every feature created by that request and report whether rollback
was complete.

## Minimal physical validation

The repository recipe `recipes/acoustic_duct_2d.py` builds a 1 m by 0.1 m
lossless air duct with rigid long walls, a 1 Pa inlet, and a zero-pressure
outlet at 100 Hz. This is below the 1715 Hz first transverse cutoff, so compare
the center pressure with the one-dimensional Helmholtz solution. One accepted
COMSOL 6.4.0.293 run used 62 triangular elements and measured
`0.8209317956 Pa` versus `0.8209317004 Pa`, relative error `1.159e-7`.

A separate unit-square Poisson acceptance used:

```text
-laplacian(u) = 2*pi^2*sin(pi*x)*sin(pi*y)
u = 0 on all four sides
u_exact = sin(pi*x)*sin(pi*y)
```

With 578 triangular elements, the nearest-center values were
`0.9999792822` numerical and `0.9999824775` analytical, relative error
`3.195e-6`. These numbers validate the exact example and runtime build, not a
portable accuracy promise. Preserve mesh, sample coordinates, equations,
boundaries, raw values, tolerance, runtime versions, and cleanup evidence.

## Save and cleanup order

COMSOL can keep a staged `.mph` locked while its model/client remains open. Use
this order:

```text
save unique staging model -> verify nonempty staging file
-> clear/remove the owned model and client -> release solver lease
-> bounded retry of Windows sharing violations -> publish final output
-> hash model and write receipt
```

Do not call `os.replace()` immediately after `jm.save()` while the model remains
open. Without explicit overwrite permission, publish through an exclusive link
or equivalent no-replace operation so a competing output is never overwritten.

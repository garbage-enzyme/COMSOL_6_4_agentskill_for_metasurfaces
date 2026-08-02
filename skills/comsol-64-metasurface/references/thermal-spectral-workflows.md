# Thermal and spectral workflows

## Contents

- Configuration preflight
- Spectral line-shape comparison
- Thermal material ledgers
- Kirchhoff and radiation evidence
- Thermal-to-optical replay

## Configuration preflight

Normalize and compare complete simulation configurations before ownership,
filesystem mutation, or COMSOL startup. Bind source and producer identities,
geometry semantics, ordered layers, material state and loss sign, incidence and
polarization, mesh dependencies, model-tree tags, solver termination, units,
and artifact chains.

Treat exact, tolerance-based, semantic, label-only, and unavailable differences
as distinct states. A matching label is not physical identity. Preview a durable
job through the same discriminated validator used by submission; a preview is
not admission, solve success, or scientific acceptance.

## Spectral line-shape comparison

Compare candidate fits only on identical raw rows, coordinate domain, response,
polarity, support, baseline, and quality policy. Use a local polynomial as a
descriptive reference and Lorentzian or Fano models only when the declared
support brackets the feature and supplies enough independent points.

Transform wavelength data explicitly when fitting in frequency, angular
frequency, or energy. Map the fitted center and half-prominence crossings back
to the original wavelength evidence; linewidths are not invariant under a
nonlinear coordinate transform.

Report residual diagnostics, support sensitivity, AIC, AICc when defined, BIC,
deltas, and descriptive Akaike weights. These compare relative model support;
they do not prove a Lorentzian decay mechanism, Fano interference, mode identity,
or publication acceptance. Retain an unresolved result when support changes the
winner or a required crossing remains unbracketed.

## Thermal material ledgers

Use a versioned caller-supplied ledger instead of embedding a material database.
Bind every state to material and sample identity, phase and fabrication state,
source, spectral and temperature validity, uncertainty, measurement conditions,
and whether each value is measured, fitted, or assumed.

Keep carrier density, mobility, effective mass, phase fraction, and discontinuous
phase states distinct. Do not smooth across a declared phase boundary. Preserve
the ledger phasor convention and convert signs explicitly for the target COMSOL
interface. A conversion preview must include exact property/function tags, units,
interpolation and extrapolation policy, table hashes, readback expectations, and
rollback requirements. Read back the applied properties and function tags before
the licensed solve.

## Kirchhoff and radiation evidence

Substitute directional absorptivity for emissivity only after verifying the exact
frequency, direction, and polarization channel is linear, time-invariant,
reciprocal, and in local thermal equilibrium. Unknown assumptions remain
conditional; a nonreciprocal channel is not covered by the reciprocal equality.

Evaluate Planck spectra with explicit wavelength, frequency, or wavenumber
Jacobians. Preserve projected-solid-angle integration, polarization basis,
coverage, extrapolation, uncertainty, source artifacts, and every gas, aperture,
optics, analyzer, detector, reference, and background kernel. Keep this
solver-free channel calculation distinct from COMSOL Surface-to-Surface Radiation.

## Thermal-to-optical replay

Use one immutable source and one strict hash-bound manifest that names the exact
thermal material state, Heat Transfer, Solid Mechanics, Thermal Expansion,
Moving Mesh, Wave Optics, study, dataset, selection, parameter, mesh, and result
tags. Reject topology change for the minimal replay route.

Run and persist these stages under one owner:

1. preflight and exact readback;
2. stationary thermal/structural solve;
3. temperature, stress, displacement, energy, frame, and mesh evidence;
4. verified Moving Mesh/spatial-frame transfer;
5. exact optical replay with raw R/T/A and wavelength synchronization.

Keep Moving Mesh inactive during the thermal/structural solve and activate it
only for transfer and the optical stage when that is the validated frame route.
Bind result evaluation to the intended study dataset explicitly. Use normal save
for the owned current model and Save Copy for checkpoints so a checkpoint cannot
silently retarget the working model.

Validate isotropic expansion against `alpha * L * delta_T`, zero-CTE and
zero-temperature-rise invariance, fixed and convective boundaries, units,
selections, material-state presence, mesh quality/inversion, parameter rollback,
source immutability, and final process/lease cleanup. Keep execution completion,
evidence completeness, and scientific disposition separate.

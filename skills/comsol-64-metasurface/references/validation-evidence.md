# Physical validation and evidence

## Contents

- Evidence states and policy separation
- Default-on formal evidence integrity
- Polarization
- Power closure and material loss
- Wavelength synchronization
- Provenance and source integrity
- Reproduction scope and stopping rules
- Peak finding and mesh convergence
- Field artifacts and visual review
- Cross-method comparison

## Evidence states and policy separation

Preserve raw measurements and assumptions separately from classification. Use
explicit states such as `measured`, `derived_from_declared_convention`,
`label_only`, `unknown`, `not_requested`, and `not_applicable`.

Only a caller-supplied, hashed policy may classify passivity, closure,
wavelength synchronization, polarization ratio, loss agreement, or mesh gates.
An evidence collector should not silently embed project thresholds.
Default one-point audit output to `evidence_only`; a successful collector is not
a project pass.

## Default-on formal evidence integrity

When the MCP exposes formal evidence guards, inspect their effective state
before relying on a verified label. Strict checks should be enabled by default;
only an explicit per-check JSON boolean `false` may disable one. At minimum,
discover and preserve the effective state of:

- outcome-contract validation;
- artifact-chain verification;
- exact summary-claim verification;
- producer/driver compatibility for resumed work.

Capability/status output should report each check, whether its source is the
default or explicit settings, a path-redacted settings fingerprint, and whether
strict verification is fully active. Unknown fields, duplicate keys, malformed
JSON, wrong types, unreadable settings, and unsupported schemas must fail closed
rather than disabling protection.

Disabling a check may allow exploration, but the affected response and receipt
must say `strictly_verified: false`, identify the skipped check, and preserve a
stable warning. Do not suppress this warning:

> Strict evidence checks are disabled; these results were not fully verified and may contain AI-generated or hallucinated content.

Re-enabling a check requires fresh deterministic verification against unchanged
artifacts. It cannot upgrade an old unverified receipt in place. If an input,
model revision, policy, artifact, manifest, producer, or driver changed, create
a new run identity.

`strictly_verified: true` means that all effective checks were enabled and all
applicable deterministic checks passed for the exact request and artifact
bytes. It does not prove that equations, boundary conditions, material data,
mesh, polarization, convergence, or scientific policy are physically correct.
Hashes prove consistency/change detection, not physics.

For formal summaries, cite exact artifact IDs, SHA-256 digests, and machine-
resolvable values such as JSON Pointers. Preserve settings, request, source,
configuration, producer/driver, and receipt fingerprints. A completed solve,
plot, screenshot, label, or plausible fitted number cannot replace missing raw
evidence.

## Polarization

Do not infer incident polarization from a port `S/P` label or field maxima inside
a resonator.

Use a lossless all-air clone and sample median or RMS `abs(Ex)`, `abs(Ey)`, and
`abs(Ez)` in a named homogeneous top-air region away from the port and structure.
Persist:

- clone/source/configuration identity;
- sample entity IDs and coordinate range;
- requested/evaluated wavelength and incidence settings;
- aggregation method and component values;
- target/transverse ratios;
- all-air R/T residuals;
- cleanup evidence.

Total field at a reflective physical port is not incident-field evidence. Some
full-field formulations expose background-field variables that evaluate to zero;
verify the formulation before relying on them.
In particular, do not assume `ewfd.Ebx/Eby/Ebz` contains an incident field in a
full-field `PeriodicStructure` model.

For angular claims, test signed elevation in both directions, at least two
azimuth paths, and both polarization labels at sparse normal/intermediate/end
angles. Match the paper/design incidence plane and physical field direction
before comparing peak motion.

### Coherent absorptivity and polarization reconstruction

Scalar RHCP/LHCP absorption alone cannot determine a polarization ellipse. For
one fixed geometry, mesh, wavelength, and incident wave vector, reconstruct a
Hermitian absorptivity matrix `H` in the Jones basis `[S,P]` from four coherent
solves. With COMSOL's calibrated incident convention
`RHCP=(S+iP)/sqrt(2)`, measure `A_S`, `A_P`, `A_Mixed` at
`LinearPol=Mixed, etaP=0.5`, and `A_RHCP`, then use:

```text
H_ss = A_S
H_pp = A_P
mean = (A_S + A_P)/2
Re(H_sp) = A_Mixed - mean
Im(H_sp) = mean - A_RHCP
H = [[H_ss, H_sp], [conj(H_sp), H_pp]]
```

If a different circular Jones-vector convention is calibrated, derive and record
the corresponding sign rather than copying the last equation. An optional LHCP
solve must satisfy `A_LHCP = mean + Im(H_sp)` within a caller-declared tolerance;
use it as an independent reconstruction check, not a fifth fitted input.

Persist every basis result independently with raw R/T/A, closure, material Qh,
requested/applied polarization properties, angle/wavelength readback, mesh
identity, and artifact hashes. Before interpreting an eigenvector of `H` as a
polarization state, require finite entries, Hermitian readback, eigenvalues inside
the caller's passive tolerance of `[0,1]`, and agreement between `A` and summed
material loss. Treat a materially negative eigenvalue as a failed reconstruction,
not a plotting artifact.

Extract the dominant eigenvector only after fixing its arbitrary global phase.
Compute Stokes parameters under the project's declared phasor, propagation, and
viewer convention; do not reuse a normal-incidence handedness sign at oblique
incidence without calibration. For reciprocal thermal emission, explicitly apply
the direction reversal and conjugation/transposition required by the declared
Kirchhoff/reciprocity convention. An incident absorptivity eigenvector is not by
itself proof of the displayed far-field radiation handedness.

## Power closure and material loss

Record raw port R/T/A without clipping. For physical closure, define caller-
declared top and bottom flux planes inside real homogeneous media, with selection
IDs, plane positions, normal/sign conventions, medium identity, expressions, and
units.

Require finite passive R/T/A and compare:

```text
A_flux = 1 - R_flux - T_flux
A_volume = integrated Qh / declared incident power
```

Cross-section absorption and volume loss may agree because they share the same
normalization. Label that comparison `internal_normalization_consistency`; never
use it as a substitute for independent physical flux closure.

Persist per-domain `Qh` for lossy materials. Treat tiny signed values in declared
lossless domains as numerical noise only under an explicit tolerance.

## Wavelength synchronization

Whenever dispersion uses `wl`, record in every row:

- requested wavelength;
- evaluated global `wl`;
- `c_const/ewfd.freq`;
- exact material expressions or their configuration hash.

Use COMSOL's exact `c_const` for the synchronization verdict. Do not substitute
the rounded convention `3e8/ewfd.freq`: it creates a systematic wavelength offset
even when the solve is synchronized. If a historical report requires that value,
store it in a separate field labeled `derived_from_declared_convention` and keep
it out of the synchronization gate. Diagnose any remaining systematic difference
before interpreting spectra.

## Provenance and source integrity

Every row or artifact should include:

- source model relative identity and SHA-256;
- normalized configuration ID;
- requested/evaluated point settings;
- physics, material, mesh, study, dataset, and selection identities;
- mesh expressions and observed element/DOF counts;
- raw R/T/A/flux/loss values and validation state;
- solve time, error, attempt, and durable timestamp;
- script/tool/schema hashes when applicable.

Never mix rows from different geometry, materials, mesh, normalization, or
selection definitions in one configuration. Re-hash the source after clone
cleanup.

Define mesh identity over the parameters that can actually change the mesh.
Wavelength can alter diffraction orders or trigger a physics-controlled rebuild,
so a single element/vertex count across all wavelengths may be an invalid gate.
In that case, put wavelength on the outer loop, persist one observed mesh
identity per wavelength, and require equality only across inner parameters that
do not affect meshing. If the workflow explicitly builds at the shortest
wavelength and reuses that mesh, key identity by geometry instead and verify the
same count after every longer-wavelength solve. Record both the policy and the
observed counts; neither pattern by itself establishes mesh convergence.

If a physically valid row fails only because an earlier mesh-identity policy was
scoped incorrectly, preserve that row and error status as diagnostic evidence.
Create a versioned driver, manifest, and output path with the corrected
dependency key; never relabel or overwrite the old row in place.

For validation matrices, each result row must bind the immutable spec, exact
point fingerprint, collector name, artifact identifier, and wrapper-manifest
hash. Keep full evidence in bounded artifacts; status and list operations return
only bounded summaries.

A successful call is not complete evidence when its manifest is partial,
missing, integrity-blocked, outside the attempt subtree, or inconsistent with
the row. Incidence metadata remains `label_only` unless typed application plus
parent-feature and port readback prove the exact setting used by the solve.

## Reproduction scope and stopping rules

Record the caller's requested evidence tier before scheduling expensive sweeps:

- For a core-mechanism reproduction, require the central spectrum or response,
  contrast between the relevant channels, passive and synchronized raw evidence,
  own-peak mesh convergence at the design point, and sparse signed branch checks
  at normal, intermediate, and endpoint settings on the relevant paths.
- For figure-by-figure reproduction, add only the continuous grids and rendered
  artifacts needed by the named figures.
- For a new or publication-grade claim, add the broader convergence, uncertainty,
  alternative-model, and continuous-domain evidence required by that claim.

Do not make a dense parameter map an automatic acceptance gate. If sparse data
already establish the caller's declared claim, stop or use an offline table or
line plot unless the caller explicitly requests continuous-bandwidth evidence or
figure matching. Conversely, never infer continuous flatness, angular extent, or
branch absence from sparse points. Treat presentation volume and scientific
necessity as separate decisions.

Have a preliminary summarizer report exact artifact identities, raw extrema,
validation failures, and unresolved boundaries. Let a separate reviewer apply
the final acceptance policy; a summary is not itself a scientific verdict.
Treat console-only or report-only numerical claims as unverifiable. If a cited
mesh, configuration, wavelength, or peak has no matching hash-bound raw artifact,
exclude that claim from acceptance rather than reconstructing evidence from the
summary.

## Peak finding and convergence

Use staged discovery:

1. Scan broadly and find every local candidate.
2. Extend if a maximum lies on or near a scan boundary.
3. Refine around each interior bracket.
4. Skip duplicate wavelengths between stages.
5. Save the branch identity and bracket evidence.

When later targets depend on completed locator or peak rows, persist the derived
target list and its configuration identity before solving that stage. On resume,
reuse the frozen list; do not regenerate it from a mixture of original and
partially refined rows, which can change the requested set and break exact resume.

For angle sweeps, continue branches from the preceding angle with adaptive
windows. Scan the dominant polarization first; evaluate the suppressed
polarization at each accepted dominant-branch peak. Run a full suppressed scan
only when it is unexpectedly strong or another branch appears.

Compare every mesh at its own bracketed peak. Record fitted center, linewidth,
Q, residuals, baseline rule, threshold, and crossings. Fano, Lorentzian,
absolute-half, and half-prominence widths are not interchangeable. Use identical
spectral support and baseline definition across meshes.

Completion of every durable row and passage of passivity, closure, and
wavelength-synchronization gates do not establish mesh convergence. When a
declared mesh-shift gate is missed marginally, first repeat the solver-free peak
fit over several reasonable neighboring-point supports. If the fitted shift
remains outside the gate, preserve `mesh_not_converged` or an equivalent
residual status. Do not add another mesh level in a reproduction task unless
the caller expands the requested evidence scope.

A fixed-wavelength amplitude difference is a diagnostic, not convergence proof.
Adding positive loss cannot increase total Q; if simulated Q is already low,
audit geometry, radiation coupling, mesh, mode assignment, and fit definition.

Run mesh bridges at normal, intermediate, and endpoint angles before generating
a dense angle-wavelength map.

## Field artifacts and visual review

Evaluate field values and coordinates from a solved dataset, validate finite
matching shapes, then select and interpolate a declared slice. Store full arrays
in a bounded artifact such as compressed NPZ and return only hashes, counts,
extrema, component ratios, ranges, coverage, and a bounded sample.

For paired on/off-resonance evidence:

- use identical slice, grid, interpolation, units, and color limits;
- report missing-cell/coverage statistics;
- preserve source/configuration/dataset hashes;
- optionally render PNGs in an isolated plotting process rather than relying on
  headless COMSOL image export.

For paper-target field maps, read the exact field-export header before choosing
the wavelength, slice coordinate, grid, or color scale. Different material
states can use slightly different wavelengths even when the caption gives one
approximate resonance, and a main panel can use per-view limits while the SI
shows the same arrays on a shared scale. Preserve both renderings when both are
scientifically relevant; compare only self-calculated arrays against the author
arrays.

Do not transfer a rotational-symmetry argument from an integrated spectrum to a
fixed-coordinate local-field profile. A local-field equivalence must transform
the physical incident polarization and the detector coordinates, line tangent,
and slice normal together. For example, C4 symmetry can imply
`|E|^2(x,0; Ex) = |E|^2(0,x; Ey)`, but not
`|E|^2(x,0; Ex) = |E|^2(x,0; Ey)`. Record the exact symmetry operation and
transformed probe geometry. Similar R/T/A under orthogonal polarizations does
not validate an unrotated field line.

When symmetry supplies uncomputed rows in a scalar spectrum or parameter map,
label them `derived_from_verified_symmetry`, not `solved`. Bind every derived row
to the source-row fingerprint and an explicit transform over geometry parameters,
polarization, incidence/detection coordinates, and sign conventions. Keep solved
and derived counters separate. Before using the transform for a dense map, verify
representative paired solves at the relevant material state and at more than one
mesh/evidence level when feasible. A sign reversal at one point does not validate
local fields or an unrelated observable.

For line-profile comparisons, report an absolute or normalized error metric
alongside correlation and preserve the normalization rule. Correlation `1`
allows an affine amplitude/baseline difference; it does not prove pointwise
identity. When identity matters, also require slope near `1`, intercept near
`0`, and RMSE within the declared tolerance on identical coordinates.

Periodic author exports can place the displayed unit-cell origin on a periodic
boundary while the reproduced model uses a centered primitive cell. Align such
maps only by a deterministic translation derived from the declared lattice
vectors and geometry, such as an exact half-period recentering. Record the
translation, wrapping rule, and pre/post coordinate ranges. Never maximize image
correlation, hotspot overlap, or another agreement metric to choose the shift;
that would tune the comparison to the target rather than reconcile coordinate
conventions.

Treat live field-tool discovery as authoritative. If an existing-dataset field
extractor rejects a bounded raw request because it requires private normalized
transport fields such as fingerprints or derived grid counts, do not invent
those fields by hand. Use a documented public solver-free normalizer when one
exists; otherwise record the contract mismatch and export the solved dataset
through direct bounded clientapi interpolation with the same hashes, grid,
coverage, and visual-review requirements. A preflight `next_call` hint is also
non-authoritative when the live profile already exposes and successfully runs
the named tool.

Numerical code may report ratios. It must not claim mode identity, symmetry,
localization, magnetic character, or publication quality. Require an image-
capable reviewer to confirm receipt of every artifact hash and return structured
observations/uncertainty. Visual review cannot override numerical policy gates.

## Cross-method comparison

Before explaining FEM/RCWA offsets:

1. Compare exact geometry bounds, dimensions, centers, materials/loss signs,
   selections, layer termination, lattice vectors, incidence, wavelength
   controls, mesh/order, and study definitions.
2. Create one immutable common baseline when any difference exists.
3. Run a small bridge matrix around all competing candidates.
4. Verify physical polarization and closure at each bridge point.
5. Refine every branch and compare each method/mesh at its own peak.

Do not let a scattered-field result with nonphysical absorption arbitrate a
physical port/RCWA disagreement.

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
- Mode-window truncation and reading stored analyses back
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

### Numbers in reports must be read, never recalled

A derived number written into reporting prose from memory or from a plausible
recollection is not evidence, and it is the hardest kind of error to notice
because it sits next to correct numbers and usually does not change the
conclusion.

A verified case: a plan document quoted a gate margin of 0.0755 per cent while the
durable artifact recorded 1.545e-05, i.e. 0.001545 per cent. The quoted figure
matched nothing in the data and overstated the margin by roughly a factor of
forty-nine. The pass/fail conclusion was unaffected, which is exactly why it
survived review for several rounds.

Rules:

- Every derived figure in a report must be **read from an artifact or recomputed
  at the moment of writing**. Never restate one from memory.
- Prefer having the report generated from the artifact mechanically, or at least
  cite the artifact field name next to the number.
- When a unit or scale is involved (fraction versus per cent, relative versus
  absolute), state it explicitly beside the value.
- Keep a **supersession register**: when a previously reported value is corrected,
  record the superseded value, the reason, the corrected value, and whether the
  conclusion changed. Never silently overwrite a wrong number, and never delete
  it — a reader must be able to see what changed.
- A corrected value whose conclusion is unchanged is still worth recording: it
  tells the reader which conclusions were never actually supported by the number
  that was quoted.

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

Q and extra loss (S02): under a *fixed mode and fixed coupling*, adding positive
material loss cannot increase that mode's pole Q. This is not a universal law
for every reported Q:

- Distinguish pole Q (complex-frequency pole), spectral-fit Q (Lorentzian/Fano
  linewidth), and branch-switching that changes which peak is labeled.
- A passive two-mode system can show a fitted Q that does not monotonically
  decrease with loss when modes hybridize or the fit window/definition changes.
- If simulated Q is already low, audit geometry, radiation coupling, mesh, mode
  assignment, and fit definition before blaming material loss alone.
- Do not use a Q-rise argument to override a verified FEM result without the
  fixed-mode fixed-coupling premise and matching Q definition.

Run mesh bridges at normal, intermediate, and endpoint angles before generating
a dense angle-wavelength map.

### Bounding discretisation when the mesh cannot be frozen

To attribute a between-configuration difference to physics rather than
discretisation, the natural design is to hold the mesh fixed. That may not be
attainable; see `troubleshooting.md`. When it is not, bound the contribution
instead of assuming it away:

1. Solve every configuration at two resolution levels.
2. Mesh sensitivity per configuration = relative change of the quantity between
   the two levels.
3. Between-configuration effect per resolution level = relative difference of the
   quantity across configurations at that level.
4. Call the effect resolved only if it exceeds the mesh sensitivity at **both**
   configurations **and** both resolutions.

Report this as a bound, never as proof that the mesh was frozen. A large,
resolution-stable effect with a small mesh sensitivity is strong evidence it is
not discretisation; it does not by itself identify the physical mechanism, and
the mechanism should be reported as open unless separately established.

An effect can also contradict the mechanism you expected. If a quantity moves in
the direction opposite to the proposed explanation, record the refutation rather
than fitting the observation to the hypothesis.

### A failed mode-window gate may be a truncation artefact

When eigenmodes are tracked over a fixed number of computed modes, the standard
guard is: no reported band may occupy one of the two highest or two lowest
positions of the computed set, because such a band may continue outside the
window. Checking that condition is necessary, but the raw count of offending
bands is easy to misread in **both** directions.

- **A raw count can overstate the problem.** Such a gate usually counts *every*
  tracked band, including irrelevant high and low modes that were never
  candidates. If the reported set is a subset, re-express the question against
  that subset — while leaving the registered verdict in place.
- **A raw count can also misattribute it.** Several limited tracks sharing a
  nearly identical endpoint frequency is the signature of **truncation**: those
  tracks do not end there physically, they were cut by the window. In a verified
  case seven tracks at one window width terminated within about 1.8 THz of each
  other; at a wider width the same tracks resolved with comfortable edge margins.

Practical procedure:

- When two window widths are already available, compare them. Outcomes of
  unrelated gates will usually be identical; what changes is *which* tracks are
  edge-limited.
- Cluster the **endpoint frequencies** of the limited tracks. A tight cluster
  indicates truncation; scattered endpoints are more likely genuine proximity to
  the edge.
- Report separately: the original verdict, the count restricted to the reported
  subset, and the endpoint clustering. Never silently replace the registry's
  verdict.
- Whether a gate should be re-registered against a narrower reported subset is an
  **authority decision**. State the question; do not answer it unilaterally.

### Reading a stored analysis back

When re-analysing a stored result file, read the **actual field names** from the
file rather than assuming conventional ones. A verified case searched for a
frequency field named like `f_THz` in a table that actually used `f_at_Gamma` and
`f_at_last`. The lookup returned nothing, the derived distance test silently
matched zero bands, and that zero was briefly read as a clean result. It was an
artefact of the failed lookup, and the correct answer reversed the conclusion:
six of nine nearby tracks were in fact window-limited, not none.

- Inspect the key list before extracting, or assert that every field you depend on
  is present and non-null.
- Treat a suspiciously clean derived result — especially a count of exactly zero —
  as a prompt to verify the lookup rather than as a finding.
- Record which field names were actually used, so a later reader can check the
  extraction instead of re-deriving it.

### Localising a tracking failure instead of reporting a pass rate

A branch-tracking gate is usually reported as a rate: how many consecutive
assignments met the overlap and margin thresholds. A rate alone cannot tell you
whether the tracker is diffusely unreliable or fails on one specific segment, and
those two cases call for completely different responses. Three cheap breakdowns
separate them.

- **Break the failures down by segment of the path**, and normalise by the number
  of assignments each segment contributes. In a verified case the bad fraction was
  roughly 16 per cent on one leg, 12 per cent on another, and 3 per cent on a
  finely sampled region near the symmetry point — a 3–5× concentration that a
  single aggregate rate hid completely.
- **Check whether the runner-up beat the chosen match.** If the stored
  per-assignment data carries a runner-up score, count the assignments where it
  exceeds the selected one. Those steps are outright **mis-assignments**, not
  weak-but-correct matches, and they warrant a different remedy. A verified case
  had steps where the chosen overlap was ~4e-04 against a runner-up of ~3e-01.
- **Repeat the breakdown at a second window width** if one exists. A failure that
  keeps the same per-segment fraction when the window changes is a property of the
  tracking; one that moves with the window is an artefact.

Report the original gate verdict unchanged alongside the breakdown. Localising a
failure explains it; it does not convert it into a pass, and it does not license a
narrower gate. Whether to re-register a gate against a subset of the path is an
authority decision.

Also state plainly what the breakdown does **not** cover: whether any failing step
lies on the specific branch you intend to report. Localising failures across the
path is not the same as clearing the branch of interest.

### Verify integrity by hashing, never by timestamp

"It was modified in the last N hours" is not an integrity test and produces both
false alarms and false reassurance.

A verified false alarm: a recursive "modified in the last 24 hours" check flagged
an entire upstream package as touched. Every file in it was in fact unmodified and
about twenty hours old — the newest was dated late on the previous day, which falls
*after* the `now - 24h` cutoff while still being untouched. The same expression
would have reported "untouched" for a genuinely modified file whose mtime happened
to fall just outside the window.

Rules:

- Verify integrity by **re-hashing against the package's own manifest** and
  comparing recorded SHA-256 values. Copying, restoring, or re-running a tool can
  change a timestamp without changing content, and can change content while
  restoring a timestamp.
- When you do need a recency check, compare against an **explicit absolute
  timestamp** captured at the start of the session, not a rolling `now - delta`
  window, and treat it only as a hint to investigate.
- Read the manifest's real structure before iterating it. A verified case assumed
  a keyed object and iterated its properties, which silently checked **zero**
  files; the resulting "all mismatched" output was an artefact of the empty loop,
  not a finding. Assert that the number of entries checked equals the number the
  manifest declares.
- Report the counts explicitly — `checked`, `mismatched`, `missing` — so a zero
  loop is visible instead of looking like a clean pass.

### Telling a mode swap from a merely weak match

When tracking fails at a step, the remedy depends on *which kind* of failure it is,
and the overlap value alone does not distinguish them.

- A step with a low overlap but a **clearly beaten runner-up** is a correct
  assignment of a rapidly changing mode. It fails a strict overlap threshold but
  it is not an error.
- A step with a low overlap and a **near-tied runner-up** is a probable **mode
  swap**: the tracker had no decisive preference and picked one of two comparable
  candidates.

The sharpest additional discriminator is the **frequency step relative to the
track's own typical step**. Compute the median absolute step along the track and
express each suspect step as a multiple of it. In a verified case one failing step
was 6.6× the median (a smooth, if large, change) while another was **66×** the
median, jumping about 2.1 THz between adjacent samples — the signature of a swap.

Two cautions:

- **Mode indices are not comparable between different runs.** Indices are positions
  within each run's own mode list, so an index change between two window widths
  means nothing by itself. Overlap values are geometric properties of the fields
  and *are* comparable; in the same verified case they agreed between windows to
  about 1e-10 while the indices differed. Use the overlaps, not the indices, to
  establish reproducibility.
- **Report the baseline you compare against.** "A large jump" is only meaningful
  as a multiple of the track's own median step, and the median must be computed
  from the track, not assumed.

Finally, localising a failure to a specific branch is not the same as clearing it.
State the failing steps, their kind, and what remains untested.

### Mode identity and overlap diagnostics

Frequency continuity alone is a weak branch identifier; pair it with a field
overlap against a reference mode, and report the runner-up and the separation.

- Take the modulus **after** summing over vector components:
  `|sum_c <Ea_c, Eb_c>|`. Taking the modulus per component and then summing loses
  the relative phase and can report a degenerate rival as a near-match.
- An unweighted common-node evaluation is a diagnostic, not a formal modal inner
  product. A formal inner product needs a fixed physical sampling or interpolation
  scheme, or the FEM mass matrix, and a stated Bloch-phase convention.
- Confirm the two field exports share coordinates, units, and point ordering
  before comparing. When configurations differ in extent, the exported point sets
  may not coincide: intersect on coordinates to recover a common-node diagnostic
  and record explicitly that this fallback was used.
- Near degeneracy: compare the subspace and assign candidates one-to-one rather
  than relying on a single pointwise maximum.
- Mode indices from the installed `mph` bindings are 1-based; index 0 selects the
  last mode, which silently shifts the whole spectrum if used as "the first".
- Read the returned solution count from the evaluation result. Do not assume the
  requested count was returned.

### Eigenvalue quantities

`ewfd.freq` exposes only the real part, so `imag(ewfd.freq)` is identically zero.
Take the decay from the complex eigenvalue or from the reported Q factor, and
cross-check the two against each other. Keep the raw complex eigenvalue in the
record so a later reader can recompute rather than trust a derived scalar.

When comparing an eigenvalue-derived Q with an energy/flux-derived Q, both must
use the **same** control volume. Using a total-domain energy against a flux
through an interior probe plane mismatches the two and produces a discrepancy
that is bookkeeping, not physics. Keep the flux sign; do not take absolute values
before comparing, since opposite-signed contributions are part of the evidence.

### Reading batched evaluations safely

`model.evaluate(list_of_expressions)` does **not** always return a flat,
one-value-per-expression array. The return shape depends on the expressions:

- A batch of **scalar-only** expressions returns a 1-D array with one entry per
  expression, in the order requested.
- Adding even **one field expression** (one that evaluates per mesh node, such as
  a field norm) changes the whole batch to a 2-D array of shape
  `(n_expressions, n_points)`, where the scalar rows are broadcast across the
  point axis.

The failure mode is silent and severe. Flattening a mixed batch and indexing it
positionally returns the **first nodes of the first field expression**, not the
expressions requested — so several different variables can appear to hold the
same value. In a verified case, a flattened mixed batch reported the frequency
for the Q factor, an energy integral, and a PML energy simultaneously.

Rules:

- Prefer **scalar-only** batches and assert `result.ndim == 1` and
  `result.size == len(expressions)` before indexing. This converts the trap into
  a loud assertion.
- If a field must be read, read it in a **separate** call, or index the 2-D
  result by row (`result[i]` for expression `i`) rather than flattening.
- Never rely on `.reshape(-1)` followed by positional indexing for a batch whose
  shape you have not asserted.
- When two or more supposedly distinct quantities come back identical, suspect
  this trap before suspecting the physics.

### Comparing oscillatory signed fields between two runs

A Poynting component or any other signed, oscillatory field cannot be compared
between two configurations by taking a per-region **mean and dividing**. Where
the mean sits near zero, the ratio is unbounded noise: a verified attempt produced
per-bin ratios of −16.5, −14.9, −10.4 and +5.2, and a physical flux ratio cannot be
negative. The median of that noise happened to look plausible (1.297) and nearly
became a finding.

Well-conditioned substitutes, in order of preference:

1. **A proper surface integral** over a real internal face, via a COMSOL
   integration operator. This is the only route to a power.
2. **Magnitude statistics** when only nodal values are available:
   `sqrt(mean(S^2))` or `mean(|S|)` per region. These stay positive and bounded.
   Report the ratio's median **and** its quartiles, and state the fraction of
   regions whose ratio falls outside a sane interval — a well-conditioned
   comparison should have none.
3. **Cross-resolution consistency.** Repeat the comparison at two mesh
   resolutions. A statistic that disagrees between them, or disagrees with an
   independently measured integral, is not usable no matter how plausible its
   median looks.

Two further cautions:

- A **nodal sum is not a surface integral** unless the mesh is uniform. Node
  density varies with the local mesh size, so an unweighted sum over-represents
  densely meshed regions. Do not report watts from one.
- **Check the cancellation level before trusting a signed sum.** If the positive
  and negative contributions nearly cancel, the residual is dominated by
  cancellation error. A verified case had a cancellation fraction of about 0.99,
  which made the signed split unusable for distinguishing two runs even though
  each run's own numbers were reproducible.
- If every available statistic fails, report the **diagnostic failure** and state
  what evidence would be required, rather than proposing a mechanism the data
  cannot support.

### Fitting a band curvature at a symmetry point

At a band extremum the dispersion is even in the wavevector, so the natural model
is `f(k) = f0 + a*k^2`, optionally with a `b*k^4` term. Three rules keep such a
fit honest:

- **A fit with as many free parameters as data points is not a test.** Two points
  fitted with a two-coefficient model give a residual of exactly zero by
  construction. Report the degrees of freedom (`n_points - n_free_parameters`)
  and exclude zero-residual-by-construction fits from any "best fit" verdict;
  otherwise the reported minimum is a meaningless 0.
- **Do not extrapolate a fit past a known channel-opening threshold.** A new
  propagating order changes the physics at a computable normalised frequency; a
  fit established below that threshold says nothing above it.
- **Report the margin, not just pass/fail.** A fit whose residual meets a
  tolerance by only a small factor is weak evidence, and a higher-order model
  fitting almost exactly usually means it is absorbing the data rather than
  testing it.
- **Label fit windows with their true point counts**, derived from the data
  rather than hard-coded, and state which window the headline number came from.
- **Report a boundary-condition variant as a control column only.** Two different
  boundary-value problems are not required to agree, so never present agreement
  between them as validation or disagreement as a failure.

When a Q changes between two configurations, `Q ~ W / P` decomposes the change
into a stored-energy factor and an outflow factor. Measure both, plus a
shape-invariant check, before proposing a mechanism:

- **Energy partition ratios** (for example the active-material fraction of the
  control-volume energy, and the magnetic-to-electric ratio) identify whether the
  mode itself changed. If these agree to several significant figures, the
  configuration did not swap or re-partition the mode, and the change must live
  in the outflow.
- **Compare the relative movement of the two factors.** Whichever moves further
  dominates; reporting only the Q change cannot distinguish "more energy stored"
  from "more power leaving".
- **Check the sign against the proposed mechanism.** A mechanism that predicts a
  smaller absorbed fraction *and* a smaller outflow is refuted if the outflow
  rises instead. Record the refutation rather than fitting the number to the
  story.
- **Fluxes through paired probe planes may add rather than cancel.** If both
  planes are oriented outward, the net outflow equals the sum of the magnitudes
  and there is no catastrophic cancellation. Verify this rather than assuming a
  small-difference measurement is ill-conditioned: compute the ratio of the net
  to the sum of magnitudes and report it.
- **Two configurations cannot establish a trend.** With two values, monotonic,
  saturating, and non-monotonic behaviour are indistinguishable. State the
  dependence as undetermined unless a third configuration is available.

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

For geometry-aware native field plots, bind the artifact to physical surface
categories, not only numeric entity IDs. Record adjacent domains, representative
face centers and normals, exposed-versus-internal classification, and the exact
selected IDs for each geometry state. Re-probe mirrored or rebuilt geometry;
an external material-air boundary, an internal material-material interface, and
a Cut Plane are not interchangeable even when their projections overlap.

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

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

### Classify "infinite" quality factors by magnitude, not by the literal

A solved cavity can report a quality factor as a literal `inf` for a truly
non-radiating mode, but it can also report a **finite number of enormous
magnitude** for the same physical situation — values of order 1e13 to 1e17 are
common when a mode has no radiation channel but the solver still returns a ratio.

A verified case counted only entries equal to `inf` and concluded that a
closed-termination path was *not* uniformly non-radiating. Classifying by
magnitude instead — `inf` **or** above a stated large threshold — gave 95 per cent
rather than 4 per cent. The physical reading was the opposite.

- Classify into explicit buckets: literal `inf`, finite above a declared large
  threshold, and finite below it. Report all three counts, not a boolean.
- State the threshold you used, since it is a reporting choice, not a measurement.
- Compare against the physically expected structure. A termination that closes the
  radiation channel *should* give non-radiating modes; if the classification says
  otherwise, suspect the classification before the physics.
- Keep the same discipline for the reverse case: do not call a large-but-finite Q
  infinite without saying so, and never derive a bound from a Q whose magnitude
  exceeds what the solve can resolve.

### Distinguish a fixed mesh along a path from a frozen mesh across states

"Mesh not frozen" and "mesh held fixed" are different claims and can both be true
in the same project.

- **Freezing across changing geometry** means two different model states must get
  the *same* sub-region mesh. This can be unattainable; a mesher may not
  reproduce a sub-region mesh across states even for provably identical geometry.
- **Holding a mesh along a path at constant geometry** means sweeping one
  parameter without altering the geometry, and one mesh is reused. This can
  succeed trivially.

Verify which one applies by reading the per-case element counts and checking
whether they are constant, rather than assuming either. Record the shared count:
a constant element count along a path is worth stating explicitly, because it means
the sweep result is not contaminated by a changing discretisation.

### Judge a tracking step against the local mode spacing

An overlap threshold cannot tell you whether a tracked band moved plausibly,
because overlap says how similar two fields are, not how far the frequency moved
relative to what the spectrum allows.

Compute, for each case along the path, the **median nearest-neighbour spacing** of
that case's own mode list. Then express each consecutive frequency step as a
multiple of the spacing at that k:

- a step **well below** the local spacing is unambiguous — no competing mode is
  close enough to be confused with it;
- a step **comparable to or above** the spacing is a candidate hop, because a band
  does not move several mode spacings between adjacent samples while remaining the
  same band.

A verified case had a median spacing of about 0.52 THz with a median step of about
0.031 THz, so most steps were safe by a factor of ~17 — while a single step reached
**9.5×** the spacing, identifying it as a hop that the overlap value alone had only
hinted at.

Two extensions worth computing:

- **Count oversized steps across all tracked bands**, not just the one of
  interest, and note how many distinct k locations they occupy. If many bands have
  one and they are spread widely, the difficulty is a property of tracking a
  crowded discrete spectrum rather than an isolated defect — which changes the
  remedy from fixing one track to reviewing the method.
- **A closed (conducting) termination concentrates the spectrum**, since there is
  no radiation channel, so tracking is hardest there. Do not transfer conclusions
  about tracking quality from a closed-termination path to an open one.

State explicitly when the comparison **cannot** be made. If one dataset records
only a single frequency per case, no local spacing exists for it and the statistic
must be reported as unavailable rather than estimated.

### Maximising overlap alone can worsen frequency continuity

When a stored tracking result includes a full overlap matrix, the assignment can be
improved offline by solving it globally instead of choosing greedily step by step.
Test whether that actually helps before reporting it as a repair.

A verified case, over a 25-point path with 16 modes per point:

| assignment | min overlap | median overlap | max step / local spacing | steps >2× |
| --- | ---: | ---: | ---: | ---: |
| greedy diagonal (original) | 0.0015 | 0.955 | 3.06 | 2 |
| optimal on overlap alone | **0.763** | 0.990 | **5.48** | **3** |
| optimal on overlap + jump penalty | 0.0021 | 0.995 | 3.06 | 2 |

Three lessons:

- **The greedy choice can be badly wrong.** Maximising total overlap raised the
  worst overlap along the path by a factor of ~500 and changed the selected mode at
  16 of 25 positions. A large fraction of the original assignment was simply
  incorrect, which the per-step overlap threshold had shown only as scattered
  failures.
- **Overlap and continuity are competing objectives.** The overlap-optimal chain
  was *worse* on frequency than the greedy one — a larger maximum step and more
  oversized steps. Optimising similarity buys similarity at the cost of
  continuity, so a repair must be scored on both criteria.
- **Neither single cost produced a usable chain.** The continuity-preserving
  variant restored the step size but left the worst overlap at ~2e-3. Report that
  as an unresolved outcome rather than presenting the best number from either
  method as a fix.

Also verify the stored matrix interpretation before relying on it: check that each
pair is square in the mode counts, and that its diagonal reproduces the
independently recorded per-assignment overlaps. A 90th-percentile ratio of best
off-diagonal to diagonal near 400 is a useful diagnostic that the existing
tracking left a lot of matching quality unused.

Re-assignment is a **re-analysis of already-solved modes**: it performs no solve,
must not overwrite the landed result, and produces a *different* tracking that
needs its own review. It never converts a registered failure into a pass.

### Separate a degeneracy failure from a threshold failure

Two tracking steps can both fail the same overlap threshold for entirely different
reasons, and the remedies do not overlap.

Inspect the **whole overlap row** for the source mode, not just the chosen value:

- **Threshold failure on a well-posed assignment.** The chosen mode has the largest
  overlap by a clear factor (a verified case: 0.79 against a runner-up of 0.21), and
  only one mode sits in a near-degenerate cluster around it. The choice is correct;
  the mode simply changed appreciably across the step. No assignment rule will fix
  this, and only a justified change to the threshold would.
- **Genuine degeneracy.** The top two candidates are close in frequency AND close
  in overlap (a verified case: 0.086 THz apart with overlaps 0.671 and 0.640, plus a
  third appreciably overlapping mode). Selecting one-to-one is ill posed here, and
  the correct treatment is to compare the 2-D subspace spanned by the pair.

Diagnostics that make the distinction mechanical:

- Count how many modes lie within a stated frequency tolerance of the best
  candidate **and** carry overlap above a stated support level. Two or more means
  the one-to-one assignment is ill posed.
- Report the runner-up ratio for the well-posed cases, so "the right mode was
  chosen" is quantified rather than asserted.
- Report the top-two overlap gap and frequency separation for the degenerate cases.

Do not merge the two into one "tracking failed" statement: a repair strategy that
addresses only one of them will look ineffective against the other.

### Verify which row of a stored matrix you are reading

Mode indices in stored tracking records are a specific convention — commonly
**0-based** — and mixing conventions silently examines the wrong mode.

A verified case hardcoded a 1-based index and then subtracted one, so it read row 8
while reporting mode 9, and printed an implausibly clean overlap (0.9988 where the
recorded value for the intended mode was 0.7905). The clean number was the tell.

- Do not hardcode a mode index. **Select the row by a physical quantity** — for
  example the recorded source frequency nearest the tracked band — and assert that
  exactly one row qualifies.
- **Cross-check against an independently recorded value.** If the record stores the
  chosen overlap for that step, require it to equal the matrix element you read,
  within tolerance. Both steps checked this way agreed exactly.
- Treat an implausibly good derived number as a prompt to verify the indexing, not
  as a result.

### Audit a persistence contract by content, not by key name

A contract that names required fields ("persist the raw complex eigenvalue, the
quality factor, the energy, the overlap with runner-up") describes **content**. An
audit that tests literal key names will report false gaps, and false gaps are
costly because they invite unnecessary rework or, worse, an unnecessary rewrite of
good artifacts.

A verified case tested six literal names against four solved point files and
reported **all four incomplete**. Mapping by content showed that five of the six
items were present under different keys — source hashes as `source_sha256_after`
plus a `source_unchanged` flag, geometry as `geometry_features`, configuration
spread over a readback block plus a termination field — and only two items were
genuinely absent.

Procedure:

- For each contract item, record **where its content lives**, and treat a
  differently-named key holding the right content as compliant. Report the mapping
  rather than a bare pass/fail.
- Distinguish a **per-item** requirement from a **package-level** one. A cleanup
  receipt, a manifest, or a claim status is normally a package artifact, not a
  field repeated in every point file; requiring it per point manufactures a gap.
- When something is genuinely missing, prefer an **additive** remedy over editing
  the existing artifacts. Editing changes hashes that the manifest already records
  and can conflict with a standing instruction not to overwrite landed results.
  Write the missing items to a separate addendum keyed by label, name the source of
  each value, and re-record the untouched files' hashes so a reader can confirm
  they did not change.
- **Cross-check rather than assert** the association. Where a label encodes
  parameters that also appear inside the file, require them to agree; that catches
  a mislabelled file without re-solving anything.

### Verify that a stored complex eigenvalue reproduces its own f and Q

When a result file persists a raw complex eigenvalue alongside a frequency and a
quality factor, check that the three are mutually consistent; otherwise one may have
been post-processed differently from the others.

Recover the convention **from the data** rather than assuming it. In a verified
case, treating the stored value as an angular frequency gave 0.0036 THz where the
file reported 66.58 THz, so that reading was wrong. The relations that held exactly
at all four points were an imaginary-part frequency and a real-over-imaginary ratio
for the quality factor, each reproducing the reported value to ten significant
figures.

- Test the candidate conventions numerically and report which one holds. Do not
  state the solver's internal convention as though it were documented.
- Report the **relative error** of each reproduction, not just agreement.
- State what this does **not** show: internal consistency of three stored numbers
  says nothing about whether the physics is right, and it does not transfer to
  other packages whose stored convention may differ.

### Do not bundle a discrimination answer with a gate status

A registered question and a registered numeric gate are different things, and a
single verdict label can silently answer the wrong one.

A verified case registered the question *"is the two-configuration disagreement
mesh or physics?"* — a **binary discrimination** question — alongside a numeric
gate requiring the two configurations to agree within 10 per cent. The delivered
verdict read `NOT_CONVERGED_effect_is_real_not_discretisation`, which answers
both at once.

Both underlying statements were true and independent:

- **Discrimination: established.** The disagreement (about 15 per cent) exceeded
  the mesh-sensitivity bound (about 0.98 per cent) at both resolutions, so it is
  physics rather than discretisation.
- **Gate: failed.** The same disagreement exceeds the 10 per cent limit, so the
  quantity is not converged and no converged value may be reported.

Reporting only the first suggests the value becomes trustworthy once discretisation
is excluded, which is false — excluding discretisation does not remove a real
dependence. Reporting only the second loses the answer to the question actually
asked, and a physically caused difference is a substantive finding rather than
merely a failed gate.

Practise:

- Keep a **two-part verdict**: the answer to the registered question, and the
  status of each registered numeric gate, each with its own status word.
- Check whether a comparison you are using as evidence is a **registered gate** at
  all. A diagnostic added to answer a question has no pass/fail of its own, and
  treating it as a gate manufactures a failure.
- When correcting a label, change **nothing measured**. Record the superseded
  string, the reason, the corrected form, and state explicitly that no value, gate
  or gate result changed.

### Check the cancellation caveat before trusting an error-bound discriminator

A common way to argue that an observed difference is physical rather than numerical
is to show it greatly exceeds the numerical sensitivity measured elsewhere. That
argument has a subtle gap and is worth stating rather than glossing.

Write each measured value as `Q(setting, mesh) = Q_true(setting) + e(setting, mesh)`.
Then

- the difference between two settings at fixed mesh contains
  `e(A) - e(B)` — a **difference of errors between settings**;
- the mesh sensitivity measured at one setting bounds `|e|` **at that setting**.

Those are not the same quantity. The relevant contaminant is the difference of
errors *between* the two settings, which can be smaller (making the comparison
conservative) or larger (making it optimistic) than the sensitivity measured within
one setting.

What can still be checked, and is worth reporting:

- **Sign and size of the mesh-induced shift at each setting.** A numerical artefact
  capable of explaining the between-setting difference would generally have to
  change sign between the settings. In a verified case the shifts were +90.4 and
  +48.0 — same sign, similar size — while the between-setting difference was about
  1370.
- **Stability of the between-setting difference under refinement.** A
  discretisation-driven difference should shrink towards zero as the mesh is
  refined. A verified case went from 1369.8 to 1412.2, i.e. it did not shrink, and
  the relative value moved only from 14.94 to 15.25 per cent.
- **Report the ratio** of the largest within-setting shift to the smallest
  between-setting difference, so the margin is quantified (about 6.6 per cent in
  that case) rather than asserted.

Then state plainly what remains untestable: the exact decomposition of the measured
value into a true part and an error is generally **not identifiable** from a small
number of points. The conclusion supported is *consistency with* the claimed origin,
not an absolute proof of it, and the possible cancellation of errors between
settings is **not bounded** by this argument.

### Verify a two-configuration comparison directly, not through a third reference

When a quantity is compared between two configurations, the comparison is only
meaningful if the same physical mode was selected in both. Two weak ways to argue
this, and one strong way:

- **Weak: the index is the same.** Mode ordering changes between configurations, so
  an index match proves nothing, and an index *mismatch* does not disprove identity
  either (a verified case selected index 15 in one configuration and 14 in the
  other for the same mode).
- **Weak: each configuration matches a third reference.** This is indirect: both
  could match a reference while the reference itself is only loosely related to
  either.
- **Strong: compare the two configurations to each other.** Intersect their
  coordinate sets and compute the overlap directly, then scan a range of mode-index
  offsets so the true partner can be found wherever it sits.

In a verified case the direct comparison gave 0.9989 and 0.9988 at the selected
indices while every other offset fell below 0.09 — a separation of about 0.92, a
factor of 12 to 14. The contrast, not the absolute value, is what makes the result
decisive.

State the limit of the method. Intersecting coordinate sets is necessary when the
two configurations have different meshes, and the intersection can be a small
fraction of each field (a few per cent in that case). The overlap is therefore a
**branch diagnostic, not a formal modal inner product**; a formal inner product
needs a fixed physical interpolation or the mass matrix.

### Comparing dict entries: exclude by value, not by identity

Selecting "the best, and then the rest" from a list of dicts is easy to get wrong
in a way that produces a suspiciously clean result.

A verified bug used an identity test to drop the best entry from a pool before
taking the maximum of the remainder. Because `max()` returns an equal but *distinct*
object, the intended exclusion silently failed and the "next best" came out equal to
the best, giving a separation of exactly **0.0**. Sorting by value and taking the
tail fixes it.

- Exclude entries by **value or index**, never by object identity, when the entries
  are dicts or tuples rebuilt during the operation.
- **Assert the aggregation agrees with the recorded best** before writing it out.
- Treat a separation of exactly zero, or any other suspiciously round result, as a
  prompt to check the selection logic rather than as a finding.

### Cross-check a thin node intersection with a binned comparison

Comparing fields between two configurations with different meshes requires a common
sample, and intersecting node coordinates can yield very few nodes — a few per cent
of each field in a verified case, because the two models did not even share an
extent. Two independent ways to check that a thin intersection is not driving the
result:

- **Restrict to the geometrically shared region.** Where the two configurations
  share geometry (a central slab that is identical by construction), recompute the
  overlap there alone. If the value barely moves, the result does not depend on the
  regions where the models differ.
- **Bin onto a common grid.** Average each field component into cells of a fixed
  size on both fields, then compare the binned fields cell by cell. This needs no
  node coincidence at all and uses far more of each field.

Reporting both, with their sample sizes, is materially stronger than either alone
because the two methods fail differently.

A caution about the binned method: its value **depends on the cell size**, and the
overlap falls monotonically as cells grow, because coarser averaging mixes distinct
structure within a cell. That monotone trend is the expected signature of averaging,
not evidence of a mismatched mode — so quote the **finest** binning as the most
faithful, and report the trend rather than hiding it.

Neither method is a formal modal inner product; state that, along with the fact that
a shared-region boundary taken from the model geometry is not re-derived by the
comparison itself.

### Reconcile mixed provenance in one result set

A result set assembled from more than one source will not share one schema. In a
verified package, two of four points were adopted from earlier checkpoints and two
were solved in the current run, and the two groups recorded **disjoint field sets**:
the adopted ones carried geometry, physics, materials and a control-volume
description; the solved ones carried the swept parameter, geometry read-backs and
domain-selection audits. Neither group's fields were a subset of the other's.

Two consequences:

- **An audit against a single key list will report false gaps.** A field genuinely
  present in one provenance group may be absent from the other, and vice versa.
  Determine the schema per group before concluding anything is missing, and say
  which group lacks it.
- **The same physical quantity may be stored with different types.** In that case
  the gap distance appeared as a bare number in one group and as a unit-bearing
  string such as `"2400[nm]"` in the other. Comparing them directly returned a null
  agreement, which reads like a disagreement but was an extraction artefact. Parse
  to a common numeric type before comparing, and keep the raw value and its type in
  the record so the conversion is auditable.

Then verify the schema difference is only a **recording** difference by pairing the
equivalent quantities and checking they agree. The strongest such checks are the
configuration invariants the analysis depends on: the control-volume domain list
and the probed face identifiers should be identical across every point if the
comparison is to be meaningful. In the verified case both were identical at all four
points while the schemas differed, which is what allowed the comparison to stand.

### Confirm an integration operator covers the same region at every compared point

When a cross-configuration comparison integrates energy over a volume or flux over a
face, the result is only comparable if the operator covers the same region in every
configuration. This is easy to leave implicit, because the operator works and
returns a number.

Record, per point:

- the **domain identifiers** each operator family covers, not merely the operator
  **tags** that exist in the model;
- the **face identifiers** used as probes, with a uniqueness check that each probe
  selection resolves to exactly one face;
- an explicit **expectation-versus-actual** record for each probe selection, so a
  selection that silently resolves to several faces is visible rather than averaged
  into the result.

Then verify across points that the control-volume domain list and the probe face
identifiers are **byte-identical**. If they are, the comparison stands on a verified
invariant rather than an assumption.

A verified package illustrated a subtlety: only some of its point files recorded the
operator-to-domain mapping, because they came from different provenance (some
adopted from earlier checkpoints, some solved in the run). The remaining checks —
that every operator family named in one schema has a corresponding operator in the
other, that the control volume agrees across both schemas, and that the probe faces
agree — all passed, but the adopted files simply did not record operator domain
coverage.

**State that as a documentation gap rather than papering over it.** The honest
statement is that everything both schemas record agrees, that the missing mapping is
not recoverable from the point files, and that the comparison is unaffected because
it consumes values the operators already produced. If the mapping matters, the
adopted checkpoint itself must be inspected. Do not claim coverage matches when the
record cannot show it.

### Check that an assigned pairing is actually its own row maximum

A mode-assignment table stores a score for the chosen pair and often a runner-up.
Those are bookkeeping values produced by the tracking step, so they can disagree with
the underlying data. Recompute from the **stored score matrix**: for each assignment,
take the source mode's row, find its argmax, and compare it with the assigned target
index.

Three outcomes, and only one is a real discrepancy:

- assigned index **is** the row argmax — nothing to report;
- assigned index is **not** the argmax — the score attached to the chosen pair is
  not the largest available for that source mode;
- the pair or row is missing from the stored data — report as unavailable rather
  than as agreement.

Cross-validate against the recorded runner-up column. Both readings should flag the
**same rows**; a disagreement means one of the two readings is wrong, which is worth
catching before either is quoted.

In a verified case, a registered near-degeneracy clause (require a subspace
comparison when the top two candidates are within a stated margin) had been applied
to only two steps. Applying it across the whole set showed it bore on 19 of 384
assignments, and the independent argmax recomputation confirmed 12 of those as
assignments whose chosen pair was not the row maximum — the worst by a margin of
0.78 in overlap. The obligation was far larger than the spot check suggested.

**Do not report such rows as a physics error.** A tracker may legitimately accept a
lower overlap to preserve continuity in another quantity — an earlier round
established that overlap and eligibility-continuity are genuinely competing
objectives. State it as a scoring outcome whose justification needs the cost
function and the competing claims, and say plainly when those are not available.

The general lesson: when a registered clause is phrased over "the two best
candidates", check it is applied to **every** row of the result set, not only to the
rows that were already under scrutiny.

### Never compare two mode windows by raw mode index

When the same sweep is solved with a different number of requested modes, the two
results are not directly comparable row by row, and a comparison that ignores this
produces confident nonsense.

A verified case compared a narrow-mode window and a wide-mode window over the same
k-steps by keying each assignment on `(step, source-mode-index)`. Because the windows
resolve **different mode sets**, the same physical mode carried a **different index**
in each — offset by 6 on one leg of the path. The comparison was therefore between
different physical modes, and its classification of "explained by truncation" versus
"persists" was meaningless.

What to do instead:

- **Align by a physical or scalar invariant**, not by index. Matching rows by their
  overlap value worked: 365 of 384 assigned values were identical to 1e-6 after
  alignment, which is what confirmed the two windows agree.
- **Report the per-leg index offset** as part of the comparison, since a roughly
  constant offset is itself the evidence that the mode sets correspond.
- **Do not compare raw counts of "bad" rows between windows of different size.** A
  wider window has more rows available, so it can show more flagged rows purely
  because it resolves more modes; that is not disagreement.

The payoff was decisive for the question actually being asked. Seeing that the weak
assignments **reproduce** in the wider window **excludes window truncation** as their
cause — if the narrow window had cut off the true partner, the wider window would
have found it and the weakness would disappear there. It did not.

Finally, be clear about scope: excluding one cause is **not** discharging the
obligation. The weak rows remained outstanding, and their justification still needed
the assignment cost, which the data did not carry.

### Fix generated summaries in the generator, not the generated file

When a package carries a hand-written summary alongside detailed artifacts, the
summary can go stale after a detailed correction. A verified case corrected a
two-part verdict in the detailed artifact and registered the supersession, but the
package's `claim_status` summary still carried the **old** conflated string, so the
two disagreed — and a reader who opens only the summary would be misled.

The first repair attempt overwrote the summary file directly. The edit **silently
disappeared** on the next build, because a consolidation script **regenerated** that
file from the detailed artifact every run.

- **Find out whether a file is generated before editing it.** If a script writes it,
  the correction belongs in the script; editing the output is temporary by
  construction.
- **Assert consistency in the generator** rather than trusting it. Refuse to write
  the summary if the recorded supersession and the artifact it summarises have
  drifted apart; a loud refusal is better than a quietly self-contradictory package.
- **Retain the superseded value under a clearly named key** instead of deleting it,
  so the change is auditable.
- **After regenerating, verify the corrected value is actually present.** The
  failure mode here is not a wrong value but a right value that got overwritten, so
  a check on the input alone would have missed it.

This generalises the earlier rule about supersessions: a correction is not complete
until every place that restates the corrected claim has been updated, and any
generated restatement must be fixed at its source.

### Sweep for every restatement before calling a correction complete

Correcting a headline verdict in its detailed artifact is not the whole job. A
package routes the same headline into several summaries — a claim-status file, a
manifest, a run receipt, a prose README — and each is a place the old value can
survive.

A verified case corrected a two-part verdict in the detailed artifact and registered
the supersession, then fixed the claim-status summary. A **sweep for the old string
across the whole package** still found it presented as a *current* verdict in the
manifest, the run receipt, and the README. A reader opening any of those would have
seen the superseded claim.

Practise:

- **Grep the superseded value across every artifact**, not just the one you edited.
  Then classify each hit: an intentional retention under a clearly named
  `*_superseded` key is fine; a hit presented as the current value is a defect.
- **Fix generated summaries in their generators** so the correction survives a
  rebuild.
- **Make the verifier cross-check summaries against the source artifact.** After
  this change the verifier asserts the manifest's verdict equals the correction
  artifact's. That is what turns "I fixed it" into "a rebuild cannot silently undo
  it".
- **Expect your own schema change to break consumers.** Turning a headline field
  from a string into an object broke the verifier that read it, and the break was
  only visible by running it. Re-run every consumer of a field whose shape you
  change.

### Sweep the numbers, not just the strings

A correction that fixes a *label* does not fix a *value*. A package can state a
fabricated or stale number in one artifact while correctly reporting it in another,
and a reader of the wrong file is misled.

A verified case had already recorded a fabricated percentage as a supersession and
corrected it **in the plan**, but the same fabricated figure survived in the
package's README and was still presented as fact there. It overstated the true value
by about **49×**. It was found only by extracting every percentage from the prose and
comparing against the durable artifacts.

Practise:

- **Extract every numeric claim from prose and check it against an artifact.**
  Percentages and quoted frequencies are the highest-yield targets. Read the
  authoritative value from the artifact — a gate-evaluation table or a results file —
  **never** from memory or from a neighbouring sentence.
- **Flag near misses, not only exact mismatches.** A number within a few per cent of
  a headline but not equal to it is the signature of a recalled rather than a read
  value. In that case the flagged figure was 0.0755 against a true 0.001545 — a
  discrepancy no human skim would catch, because both look like plausible small
  percentages.
- **Quote the superseded value only inside a correction note**, and say explicitly
  that it is the old text. Then a later sweep can tell intentional retention from a
  live defect.
- **Re-run the sweep after fixing**, and confirm the corrected value is present and
  the old one survives only in the correction note.

Keep the sweep in the package as an artifact, so the check is repeatable rather than
a one-off review.

### Route a numeric consistency check by field name, never by magnitude

Checking that restated numbers agree across artifacts looks like a job for "compare
every number in file A against every headline value". It is not, and a first attempt
at it produced **31 false positives out of 31 reported hits**.

The reason is that magnitudes collide. In a verified case a swept parameter near
`0.1` fell within 50 per cent of a gap effect near `0.15`, and the two gap effects
(`0.149` and `0.152`) fell within 50 per cent of each other. Every one was flagged as
a suspicious near miss, and none was a defect.

What works instead:

- **Route each field to the one authority its NAME restates.** Map a JSON path leaf
  (`effect_low_res`, `sensitivity_bound`, and so on) to the single source that
  defines it, then compare only that pair. The same data then yields **8 comparisons
  and 0 disagreements** — a result a reader can act on.
- **Compare magnitudes only as a fallback for unnamed values**, and expect that
  fallback to be noisy. If a check reports mostly false positives, the check is
  wrong, not the data.
- **Record why the naive version failed**, in the artifact itself. A future reader who
  sees only the corrected output cannot tell whether the quiet result means agreement
  or an over-broad filter that matched nothing.

A useful sanity property: a sweep whose hit count is implausibly high, or whose
"mismatches" are all quantities of similar size but different meaning, should be
treated as a broken test rather than a finding.

### Cross-check tabular summaries against the structured files they restate

A package commonly restates the same results twice: once as structured records and
once as CSV tables. If the two are produced by different code paths they can drift,
and the CSV is often what a reader or a downstream script actually consumes.

Verify each table row against the record it summarises, field by field, from the
authority rather than by eye. A verified case compared 48 values across a per-point
table and two derived tables with **0 disagreements**.

Two differences are **not** defects, and should be reported as such rather than
counted as mismatches or silently coerced:

- **Presentation rounding**, where the table carries fewer digits than the record.
  Compare with a stated relative tolerance rather than exact equality, and record the
  tolerance used.
- **Flattening**, where a nested record field becomes a differently named column.
  Map the names explicitly; an unmapped pair is a naming difference, not a
  disagreement.

Keep the three outcomes separate — agreed, naming difference, value disagreement —
so a clean result is distinguishable from a check that matched nothing. A check that
reports zero comparisons is not a pass.

### Compute figure annotations from the data, and expect the text to grow

Figure titles and axis labels are read more casually than prose, so a number typed
into one is a high-risk place for a stale or invented value — the same class of
defect as a fabricated percentage in text.

In a verified case one figure title carried hand-typed mesh-sensitivity numbers while
every other annotation read from the artifact. The hand-typed values happened to be
**correct**, which is precisely why the pattern is dangerous: the same habit produced
a wrong value elsewhere. Both were changed to compute their text at run time, and the
computed titles then reproduced the artifact exactly.

Practise:

- **Derive annotations from the artifact**, using an f-string over the same variable
  the plot uses. A number that cannot drift is better than one that happens not to
  have drifted yet.
- **Audit which annotations are computed and which are literals.** Parse the plotting
  script and classify each title and label. Two categories are acceptable as
  literals: **registered thresholds** (fixed parameters, which should still be
  cross-checked against the preregistration) and non-numeric text.
- **Re-render after switching to computed text.** Derived values are longer than the
  approximations they replace: a title that fitted as `~15 %` overflowed the axes
  once it became `14.9 % / 15.2 %`, clipping the conclusion. Wrapping to two lines
  fixed it — but only looking at the rendered image revealed it.
- **Check the figure, not just the script.** A successful render with clipped text
  passes every programmatic assertion.

### Get the direction right when rebuilding ratios

Before concluding that a stored factor is wrong, check that your rebuild uses the same
**direction** as the original. Ratios of a falling and a rising quantity are
reciprocals of each other, and mixing them produces a large apparent discrepancy that
is entirely your own error.

A verified case recorded a quality-factor factor as `Q[end]/Q[start]` — a number below
one, because the quantity falls. A first rebuild computed `Q[start]/Q[end]` and
disagreed with the artifact by about **38 per cent**, which looked like an artifact
defect. Correcting the direction made the rebuild agree to machine precision.

Procedure:

- **Establish which end is which** from the stored array order and the known
  direction of the physical change, then state that in a comment so the next reader
  is not left to infer it.
- **Rebuild every factor independently** and compare each with the recorded value.
  In that case two of the three rebuilt exactly and only one disagreed — a strong
  hint that the odd one out was a direction error rather than a bad record.
- **Distinguish "the raw values reproduce" from "the identity closes".** These are
  different questions. The identity `A = B/C` closes to machine precision only if
  the three factors come from one route; if the numerator comes from an independent
  method it will close only approximately, and the **residual is itself a result** —
  it measures the agreement between the two routes. Report it as a number rather than
  as a pass or fail.
- Do not label the combined outcome with a single boolean. A field named for
  "closure" that is false while every raw value reproduces exactly reads as a failure
  and misrepresents the finding.

### Refit over every defensible window before quoting a fitted parameter

A single fit that passes a residual gate is not evidence that the fitted parameter is
a physical constant. If the value swings with the fitting range, it must be quoted
together with that range.

A verified case fitted a band-curvature coefficient over all windows of landed points.
Across every window with at least one degree of freedom:

- the **sign was stable** — negative in all of them — so the qualitative content held;
- the **value moved by about 10 per cent** between windows (roughly −2.71 to −2.47);
- windows confined to the smaller wavenumbers fitted with residual of order
  `1e-6`–`1e-5`, while any window reaching the largest wavenumber was **about 20×
  worse** at `1e-4`, and the fitted value shifted systematically with it.

Reporting only the smallest window's value would have presented a range-dependent
number as if it were a constant. The honest statement is a **range**, plus the
observation that the systematic degradation is evidence the fitted form does not
describe the full range.

Practise:

- **Enumerate every window** with at least one degree of freedom, not the one window
  that looks best.
- **Report sign stability separately from value stability.** A stable sign is real
  physical content even when the magnitude is range-dependent; conflating them
  discards a defensible finding.
- **Look for systematic, not random, residual structure.** Residuals growing with the
  fitting range signal a missing term, which no amount of averaging will fix.
- **Exclude zero-degree-of-freedom fits from the judgement**, and count them
  explicitly, since they have zero residual by construction.
- **Beware a tolerance tighter than float representation.** Comparing a stored `0.1`
  (actually a value slightly above it in binary floating point) with a `1e-12` tolerance matched **nothing** and
  silently returned nulls; the resulting empty result looked like "no windows found"
  rather than a filter bug. Assert that a selection is non-empty.

### Distinguish "fits better" from "predicts better" when comparing model forms

Adding parameters always improves a fit, so a residual comparison between forms of
different size proves nothing on its own. Two checks make the comparison meaningful.

**Compare at equal parameter count.** If one family beats another at the *same* number
of parameters, the advantage is the functional form, not extra freedom. A verified
case compared even-power forms against a form linear in the absolute value of the
coordinate: the even-power family won at 2 parameters, at 3 parameters, and at 4 — a
consistent, like-for-like result establishing that the underlying feature is
**smooth and symmetric** rather than having a cusp.

**Test held-out prediction.** A form that merely interpolates cannot predict a point it
has not seen. Leave-one-out is the decisive check. In that case the simpler registered
form had a maximum held-out error of `7.2e-03`, while adding one even-power term
reduced it to `1.9e-03` — a factor of about `3.8` from **one** extra parameter, which a
form whose only advantage is freedom cannot buy.

**Then apply the same dof reasoning to the test itself.** Leave-one-out has a trap: a
form with as many parameters as the remaining training points has **zero degrees of
freedom** in every fit and therefore reproduces each held-out point *exactly*. That
looks like perfect prediction and is pure interpolation. In the verified case the
four-parameter form showed the best held-out error of all (`1.5e-04`) yet had
`dof = 0` per fit; excluding it left the three-parameter form as the genuine winner.
Compute and report `dof` per held-out fit and exclude the zero-dof forms from the
ranking explicitly.

Report the robust statements and the unresolved ones separately: that the simpler form
is inadequate over the full range, and that one family beats another at equal
parameter count, can both survive while the *precise* number of terms needed remains
undetermined by the available points.

### Check whether one point carries the whole fit

Before reporting that a richer model form is "needed", establish that the improvement
is not carried by a single sample. A small, unevenly spaced grid can make one point
decisive without any hint in the aggregate residual.

Leave-one-out on the **parameter of interest** — not on the fitted values — is the
direct test. A verified case found:

| point removed | shift in the fitted coefficient |
| --- | --- |
| smallest wavenumber | +0.0071 |
| second smallest | +0.0057 |
| middle | −0.00004 |
| second largest | −0.0008 |
| **largest** | **−0.184** |

The largest-wavenumber point moved the coefficient by **7.4 per cent**, while removing
any other point moved it by at most **0.29 per cent** — a dominance ratio of about
**26**. The design also degraded fast with model size: the condition number rose from
`6.6e1` for the base form to `8.8e3` with one extra term and `3.9e6` with two, and the
base form already had a maximum leverage of `0.955`.

That does not invalidate the finding, but it **changes what may be claimed**:

- **Survives:** statements about *residuals* ("the simple form does not reach the
  accuracy over the full range that it reaches over the narrow range") and comparisons
  made **at equal parameter count** — the same points appear on both sides, so one
  influential point affects both families and cannot manufacture the difference.
- **Weakened:** any claim that a *specific extra term is required*. A real missing term
  and one point's influence are both consistent with the data, and the available points
  cannot separate them.

Report the dominance ratio, the condition numbers, and the maximum leverage alongside
the conclusion, and state the surviving and weakened claims separately. When a
conclusion rests on one point, say so and quote the parameter with that sensitivity
attached rather than to spurious precision.

### Leverage sits on the endpoint, so trimming the range does not remove it

A tempting remedy for "one point dominates the fit" is to drop that point and refit the
narrower range. It usually does not work, and the reason is structural.

In a verified case the full five-point set had its maximum leverage at the largest
wavenumber (`0.955`), and leave-one-out showed the fitted coefficient moving `7.4` per
cent when that point was removed. Dropping it should have helped. It did not: the
narrower subset had a **higher** condition number (`2.48e2` against `6.60e1`).

Leverage per point showed why:

| point | leverage, narrow subset | leverage, full set |
| --- | --- | --- |
| smallest | 0.41 | 0.30 |
| … | 0.37, 0.26 | 0.29, 0.26, 0.20 |
| **endpoint** | **0.958** | **0.955** |

The `0.95`-plus leverage sits on **whichever point is at the end of the range**.
Removing the far point does not remove the leverage; it transfers it to the new
endpoint. And the full set is nevertheless **better** conditioned, because conditioning
depends on the **spread** of the design across the range, not only on the number of
points — the endpoint extends the span even though it carries the leverage.

Practise:

- **Attribute leverage to the point that carries it**, by reading the diagonal of the
  hat matrix per point, rather than to the point whose removal changed the fit. In the
  verified case these were different questions with different answers.
- **Report both the condition number and the maximum leverage** for each range
  considered; the two can move in opposite directions.
- **Do not present a narrower range as a fix** unless its conditioning is actually
  better. State the coefficient for each range together with its single-point
  sensitivity, and let the difference between ranges speak.
- Remember that a **true observation can support a wrong inference**. The
  leave-one-out fact was correct; the remedy drawn from it was not, and only the
  follow-up measurement exposed that.

### Establish whether a surface-flux sum is a difference or a sum of magnitudes

Before treating a net flux through a control volume as a cancellation, **compute what it
actually is**. The two cases look identical in a plot and have opposite implications
for conditioning.

A verified case had two probe planes carrying opposite-signed powers of nearly equal
size. The natural reading was "the net is a small difference of large numbers, so the
derived quantity is numerically fragile". That reading was **wrong**: the stored net
equalled the **sum of the magnitudes** of the two planes to machine precision at every
point, so the derived quantity was well conditioned and reproduced from its definition
exactly.

Include the explicit test in the record:

- compare the stored net against `|plane_1| + |plane_2|` and against
  `|plane_1 - plane_2|`, and state which it matches;
- report the **normalised imbalance** `(p1 + p2) / (|p1| + |p2|)`, which is the
  meaningful small quantity when the planes oppose — in that case between `0.24` and
  `0.87` per cent.

The imbalance is also physically informative, and reporting it changes an
interpretation. Across a configuration change the individual planes moved by `36`–`39`
per cent while the imbalance moved by only `0.27`–`0.52` percentage points, **and not in
the same direction at the two resolutions**. The quantities that move are dominated by
the **common** part of the two planes; the imbalance, which distinguishes a closed from
an open mode, barely moves. That is consistent with the change altering how much flux
*circulates* rather than how closed the mode is — a different statement from the one
the raw plane change suggests.

Report a trend that disagrees between resolutions as an unresolved observation rather
than picking the resolution whose direction you prefer.

### Design a check that could fail, then report it as failed-to-falsify

An interpretation that cannot be contradicted by any measurement is not carrying weight.
Before building on a physical picture, state a prediction it makes and test it — even
with very few points, because a **falsification** needs far less data than a
confirmation.

A verified case had an interpretation ("the configuration change is dominated by
circulating flux rather than by how closed the mode is") and derived a prediction from
it: the disagreement between two independent evaluation routes should **grow** with the
flux imbalance. On four points the correlation came out `+0.70`.

That number means almost nothing on its own, so the check was itself checked:

- **leave-one-out on the correlation**: dropping each point in turn gave `+0.57`,
  `+0.98`, `+0.44`, `+0.71` — always the same sign, so the correlation is not carried
  by one sample;
- the honest label is therefore **"survived the test"**, not "supported". With four
  points, a correlation spanning `0.44` to `0.98` under point removal carries no
  mechanism and should not be used quantitatively.

Record the reasoning explicitly:

- **State what a failure would have looked like** — here, an anti-correlation would
  have undermined the interpretation before anything was built on it. A check whose
  failure mode is unstated is usually a check that cannot fail.
- **Separate "not contradicted" from "supported".** The value of the exercise is that
  the picture was exposed to a real chance of being wrong, not that it survived.
- **Do not promote a small-sample correlation to a mechanism.** Report the sample size,
  the leave-one-out spread and the resulting limit on the claim, in the same record.
- Correlations are especially vulnerable to single-point control, so apply the same
  leave-one-out discipline used for fitted parameters.

### Establish the orientation of a batched evaluation array before indexing it

A batch of expressions evaluated over many solutions is stored as a **2-D array**, and
whether it is expression-major or solution-major decides which number you read. Getting
it wrong can return zeros rather than an error.

A verified case stored 25 expressions over 32 eigenmodes. The array was
**expression-major** — 25 rows of 32 — so `stored[expression_index][solution_index]`.
A first script assumed solution-major and indexed `stored[solution_index]`; an
assertion comparing the slice length against the expression count **failed immediately**
with a `32`-versus-`25` mismatch. Without that assertion the wrong slice would have been
read silently, because a length-32 row is a perfectly valid array.

The dangerous part is the **imaginary** component. In that data only one of the 25
expressions — the complex eigenvalue — has a nonzero imaginary part; the other 24 are
real-valued and their imaginary rows are identically zero. So an index or orientation
error lands on zeros and produces a damping of zero, which looks like a lossless result
rather than a bug.

Practise:

- **Print the shape and both dimensions** before extracting anything, and assert them
  against the known counts.
- **State the layout explicitly** in the script and in the artifact.
- **Check which components are structurally zero** and record it, so an accidental
  zero is recognisable as an error rather than as a result.
- **Then verify the recovered convention across every mode, not just the working one.**
  In the verified case the relation between the stored eigenvalue and the reported
  frequency and quality factor held for all `128` mode slots with a maximum relative
  error of `2.2e-16` — machine precision — which is far stronger evidence than
  agreement at one selected mode.

### Label a trend from the sweep variable, not from the array order

A trend label is easy to invert, and an inverted label is worse than no label because it
reads as a finding. Check the printed numbers against the stated direction every time.

In a verified case a quantity was sampled at two values of a sweep variable, giving
`7.72e-05` and `6.58e-05`. The comparison was written as "rises when the first is less
than the second", which labelled a **fall** as a **rise**. The numbers were correct; only
the word was wrong, and the word is what a reader carries away.

Practise:

- **Name the endpoints in the artifact**, not the indices — store `value_at_start`,
  `value_at_end`, and `change_from_start_to_end`, so the direction is unambiguous.
- **Say which way the sweep variable moves** when naming the change, e.g.
  `direction_as_sweep_increases`, since "the change" alone does not define a sign.
- **Cross-read the printed table against the label** before believing either. In that
  case `7.72e-05 → 6.58e-05` plainly falls, which is what exposed the error.
- Distinguish **the direction of a ratio from the direction of the underlying
  quantities**: a falling energy fraction is not the same statement as a falling
  absorbed power, and a mechanism argument must use the quantity it actually invokes.

A related caution: a trend that reproduces across an independent axis is worth more than
the absolute values that produced it. When the same fall appeared at both mesh
resolutions, the **direction and rough size** became the reportable result, while the
individual values remained subject to the unresolved convergence question.

### Rank candidate mechanisms by what tracks the observable, not by what moves most

When several quantities all change under a parameter sweep, the largest change is not the
driver. The question is which quantity the observable **tracks**.

A verified case had a quality factor falling about 15 per cent across a sweep. Three
candidate quantities also changed: a boundary-region energy fraction (`14.8` / `14.2` per
cent), a control-volume energy (`17.6` / `16.7` per cent) and a net outflow power
(`38.4` / `36.2` per cent). The largest mover was the outflow, and an earlier round had
named it the driver on that basis alone. Comparing **fractional changes against the
observable's fractional change** gave:

| candidate | mismatch, setting 1 | mismatch, setting 2 |
| --- | --- | --- |
| boundary-region energy fraction | **1.1 %** | **7.2 %** |
| control-volume energy | 18.0 % | 9.5 % |
| net outflow power | 157 % | 137 % |

The **smallest** mover tracked the observable best, and the largest mover tracked it
worst. Moving most is not evidence of driving.

Practise:

- **Score candidates by agreement with the observable's change**, not by the size of
  their own change.
- **Include control quantities that should NOT track it.** In the verified case a
  composition fraction and an energy ratio were both gap-independent, so their mismatch
  to the observable was about `99.9` per cent. That they *failed* to match is what gives
  the comparison discriminating power; without such controls, "everything matches
  everything" cannot be excluded when all candidates are of similar magnitude.
- **Report rank correlation across all points** in addition to the endpoint comparison,
  and say that an endpoint-only match is not a trend.
- **Show the raw numbers next to any mismatch metric**, so the metric is not doing the
  work unexamined.
- **Demote the previous favourite explicitly.** A reversal should be recorded as a
  reversal, naming what was withdrawn, rather than quietly restating the conclusion.

### Check whether a ratio's tracking comes from its denominator

A ratio can appear to track an observable when only its **denominator** is doing anything.
Always decompose a candidate ratio into its parts before treating its tracking as
evidence about the numerator.

A verified case found a boundary-region energy **fraction** tracking a quality-factor
change to within `1.1` and `7.2` per cent. Taken at face value this promoted the boundary
region as the likely mechanism. Decomposing it:

| quantity | change, setting 1 | change, setting 2 |
| --- | --- | --- |
| quality factor | 14.94 % | 15.25 % |
| boundary-region **energy** | **0.277 %** | **0.208 %** |
| total energy | 17.66 % | 16.74 % |
| boundary-region **fraction** | 14.78 % | 14.16 % |

The numerator was essentially **constant**; the fraction's tracking was inherited
entirely from the denominator. Reported correctly, the finding says nothing about the
absorber at all — the total stored energy carries the change.

What exposed it was ranking **every** gap-varying quantity rather than a hand-picked
list, which placed the boundary-region energy at `98.4` per cent mismatch and the
fraction at `4.1` per cent. Practise:

- **Rank all candidate quantities**, not a selected few, so the candidate list cannot be
  accused of being chosen after seeing the numbers.
- **Decompose any ratio that ranks well** into numerator and denominator and report each
  separately.
- **Exclude circular candidates.** Two routes to the same quantity will track each other
  by construction; a quality factor computed from a flux will top any ranking against a
  quality factor computed from an eigenvalue. Name the exclusion explicitly.
- **Keep a group-level statement when the individual promotion fails.** In that case the
  energies as a *group* genuinely tracked the observable far better than the surface
  powers did (which were off by `135`–`162` per cent), and that survived the correction.

### Guard a generated claim against the corrections that invalidated it

When a conclusion is revised, fixing the **output file** is not enough if that file is
generated: the next regeneration restores the old text. Fix the **generator**, and make
it refuse rather than emit a claim that contradicts the correction record.

A verified case had a summary claim naming one quantity as the driver of an effect. Two
later rounds demoted that quantity and then withdrew the promotion of its replacement.
The claim string had been typed into the generator, so it kept reporting the superseded
position for two rounds while the correction artifacts said otherwise. Editing the output
file alone would have been reverted on the next run.

The fix has two parts:

- **Derive the claim from the correction artifacts** rather than restating it as text —
  here the driver field and a candidate ranking were computed from the analysis files.
- **Assert the corrections are present and mutually consistent** before writing. The
  generator now checks that the specific supersession entries exist and that the
  correction artifact still supports the withdrawal, and raises if not:

  > `AssertionError: mechanism supersessions missing from the register: [...]`
  > `refusing to write a driver claim that may predate the corrections`

Verify the guard rather than trusting it: temporarily remove the supersession entry and
confirm the generator **fails** instead of writing. Then restore it, run the generator
twice, and confirm the corrected value survives both runs — a guard that fires but is
never tested is a guard that may not fire.

This applies to every generated summary: claim ledgers, READMEs, manifests, figure
titles. Any of them can silently reassert a withdrawn conclusion.

### Prove a passing check can still fail

After fixing a defect that a check detects, the check reports clean. That clean result is
worthless unless the check can still **fail** — otherwise "zero findings" may mean the
check was quietly weakened into always passing.

This applies with force after you edit a detector. In a verified case a sweep for stale
claims reported zero after a fix, but the sweep's own backing test had just been
rewritten (its first version produced false positives by matching a string prefix). The
same pass that fixed the false positives could in principle have made everything match.
So the detector was tested directly:

- **sabotage a copy**, not the real artifact — reintroduce the exact string the
  corrections withdrew, in a temporary copy;
- **run the detection logic** against the copy and confirm it reports the finding;
- **assert the real artifact's hash is unchanged** afterwards, so the self-test cannot
  itself have altered the thing it verifies;
- **delete the copy and assert deletion**, and record all of this in the artifact.

The result: the detector fired on the withdrawn string, so the clean result on the real
generator was a genuine pass. Had it not fired, the clean result would have been
discarded rather than reported.

Generalisations:

- **Every "no findings" result depends on the detector working.** For a check whose
  output is the basis of a claim, include a positive control.
- **Keep the control in the artifact**, not in your head, including whether the real
  input was left untouched.
- A check that was just edited deserves the control most, since the edit is what could
  have broken it.

### Read a manifest's container type before concluding it is empty

A manifest keyed by path is a JSON **object**; one keyed by position is an **array**. The
two look alike in a listing and behave completely differently under a shell.

In a verified case a shell probe of a package manifest reported **0 declared files**, and
a follow-up listing reported several analysis artifacts as "outside the manifest". Both
were **wrong, and produced by the same mis-parse**: the field was an object with one
entry per path, so counting it as a list returned a small number and iterating it as a
list yielded nothing. Read correctly, the manifest declared every file, and the only
undeclared items were the two outputs that **cannot declare themselves** — the manifest
and a receipt written after it is finalised.

Practise:

- **Check the container type explicitly** (`isinstance(..., dict)` versus `list`) and
  branch on it, rather than assuming one shape.
- **Assert the undeclared set is exactly what you expect**, naming the self-referential
  exemptions, so a real omission fails instead of blending into the exemptions. A
  manifest and any receipt finalised after it legitimately exclude themselves.
- **Prefer the artifact's own count field** as a cross-check on your parse: if the stored
  count and your parsed count disagree, suspect the parse before the data.
- **Record an apparent gap once it is explained**, so it is not re-investigated by the
  next reader — and state the reading that was wrong, since the wrong reading is what
  invites the repeat.
- A shell tool that silently coerces a single-element collection is a recurring source of
  this error; when a count looks impossibly small or impossibly complete, re-derive it in
  the same language that wrote the file.

### Reconcile the acquire and release receipts, and explain any imbalance

Counting resource-acquisition records against release records is a cheap check that finds
one specific failure: a lease or handle left held because a run died. Do it by comparing
matched name sets rather than by counting, so you learn **which** acquisition is
unmatched.

A verified case had eighteen acquisition receipts and seventeen release receipts. That
imbalance is exactly the signature of a held lease, so it was explained from artifacts
rather than noted:

- the unmatched acquisition was a diagnostic whose console log **stops mid-way through
  its case list** — one case header printed, no result after it;
- its lease record named a process that has since exited;
- the run receipt recorded that an orphan was encountered, that identity was confirmed by
  command line and lease record, that the process was terminated, and that the lease was
  reclaimed **through the sanctioned recovery path rather than by deleting a lock**;
- no external solver processes remained, and no live lease was present.

Therefore the imbalance was the known abort, and the recovery was documented.

Practise:

- **Compare matched sets, not totals.** "18 versus 17" tells you something is wrong;
  "the unmatched one is the aborted run" tells you what.
- **Assert the imbalance equals the known exception.** A future imbalance of the same
  shape must fail the check rather than be absorbed into the explanation. Name the
  expected unmatched entry explicitly.
- **Cross-check the abort from at least two independent places** — here the truncated
  console log and the cleanup block of the run receipt agreed.
- **Confirm the end state, not just the action**: no live lease file and no remaining
  processes, so the recovery is verified rather than merely logged.
- **Prefer the sanctioned recovery path** over manually removing a lock file, and say so
  in the record — the mechanism matters when the same situation recurs.

### A missing field must fail loudly, not read as a failed measurement

Reading a quantity from the wrong key yields a null, and a null fed into a threshold test
produces a **failing result that looks like a measurement**. This is the most dangerous
shape of lookup error: it inverts a pass into a fail while appearing to be data.

In a verified case a gate on a minimum overlap was recomputed by reading the overlap from
a plausible-but-wrong location under the selection block. The read returned nothing, and
the gate reported **FAIL** at both settings. The overlap actually lived two levels deeper,
under an overlap block; read correctly, the gate **PASSED** at both. The spurious failure
would have been recorded as a finding.

Practise:

- **Assert presence immediately after every retrieval**, naming the path, so a miss stops
  the script instead of propagating:

  > `assert ovl is not None, f"{label}: overlap missing"`

- **Never let a null reach a comparison.** Guard the value, not the comparison: a
  comparison that treats null as "below threshold" will silently manufacture failures.
- **Distinguish the three outcomes explicitly** — measured pass, measured fail, and
  unavailable — and refuse to emit the first two when the third is the truth.
- **Read thresholds from the registration file rather than restating them**, including
  the **comparison direction**, so a metric registered as a lower bound is never checked
  as an upper bound. Restating a threshold is how a gate gets relaxed by accident.
- **Recompute every gate from raw stored values and diff against the package.** In the
  verified case all four gates reproduced with **zero** discrepancies, which is the
  result that makes the recomputation worth having: it converts "the package says" into
  "the package and an independent reading agree".

### Verify a sweep's own matcher before trusting its findings — or its silence

A source-scanning sweep reports both the defects it finds and, when it finds none, a clean
result. Both outputs depend on the matcher working, and a broken matcher produces the
**clean** result. Four successive versions of one sweep were wrong before it was right:

| version | defect in the sweep | reported |
| --- | --- | --- |
| 1 | tested every path against one file | 83 of 98 "missing" |
| 2 | global handle→file table, but handle *names are reused across scripts* | 13 of 88 "missing" |
| 3 | patterns anchored with `^` compiled **without the multiline flag**, so nothing after line 1 matched | **0 checked, 0 found** |
| 4 | counted write targets (`d["k"] = ...`) as missing reads | 1 false finding |

Only the fourth version was correct, and it needed an assertion to get there.
Lessons, each earned by a wrong run:

- **Assert the sweep actually inspected something.** `assert checked > 50` turned the
  version-3 silence into a loud failure instead of a clean report. A sweep that examines
  nothing must never report clean.
- **Anchor patterns with the multiline flag when scanning whole files.** Without it, `^`
  matches only the first line and every subsequent match is silently lost.
- **Resolve names in the scope they belong to.** A handle name is not bound to a file:
  the same short name may be reused by many scripts for different documents, so a global
  lookup table tests paths against the wrong document.
- **Separate reads from writes.** An assignment target `d["k"] = ...` creates the key and
  can never be "missing".
- **Verify against a known-bad input.** Inject a deliberate typo and confirm the sweep
  reports it. Do this *after* the sweep reports clean, since that is the result at risk.
- **State the coverage limit the self-test reveals.** The injection here was **not**
  detected, because handles assigned from templated filenames are skipped — and those are
  the majority. The honest conclusion is "every literal-named handle resolves", not
  "every read path resolves".
- A sweep whose false positives outnumber its true findings is worse than no sweep: it
  manufactures work and, if acted on, would have "fixed" correct code.

### Expand a filename template from the labels the code uses, not from a disk pattern

To check reads written as `OUT / f"case_{label}.json"`, the obvious approach is to match
the template against filenames present. It over-matches. In a verified case the pattern
`a case-numbered pattern` also matched `case_A_raw_suffix.json`, because the
non-greedy group simply swallowed the extra suffix, producing **56 spurious findings** of
the form "a point-file field is missing from a raw-integrals file" — true, and entirely
meaningless, since no handle reads a raw-integrals file under that name.

Expand from the labels the script **actually iterates** (its `for` loops and label lists)
and confirm each resolved file exists. That removes the guesswork about which files a
handle could refer to.

### Treat a shadowed variable name as a latent result change

A name rebound to a different object mid-function can make a comparison read the wrong
dictionary. Whether it currently *works* depends on whether the two objects happen to
share keys — which is exactly what makes it dangerous.

In a verified case a summary loop rebound `b` from a loaded artifact to a row of its own
output dict, and a later comparison read `row["plane_power"]` from the rebinding. It ran,
because the row carried the same key. Before touching it:

- **Re-derive every affected value from the source data** and confirm it reproduces. In
  that case all four values matched to 12 decimal places, establishing a code-clarity
  defect rather than a data defect.
- **Fix the name, then re-run and diff against the pre-fix artifact.** Here the output was
  byte-identical, which converts "the fix is safe" from an expectation into evidence.
- **Establish correctness before refactoring**, so the fix cannot quietly change a result.

A shadowed name that happens to work is a trap for the next edit, not a wrong answer
today — and both facts deserve recording.

### Audit where a figure's DATA comes from, not only its labels

Deriving titles and axis labels from artifacts covers the text. The plotted arrays are a
separate path: a curve can be drawn from a hardcoded list that matches the data today and
drifts tomorrow, while every label remains correct.

A verified check parsed the plotting script's AST and classified the data argument of
every drawing call:

- **traceable** if it references an artifact the same script loads;
- **hardcoded** if it is an inline numeric list of length three or more with no artifact
  reference.

Result: 13 drawing calls, four artifacts read, and **zero** hardcoded series. A bare
scalar threshold passed to a horizontal or vertical reference line is acceptable — it is
a registered limit, not a data series — so the rule targets **lists**, not scalars.

Verify the classifier on an injected case:

- **insert a hardcoded three-element series into a probe copy** of the plotting script and
  confirm the check reports it, naming the line and the argument;
- **assert the real script was not modified**;
- **record the self-test in the artifact**, so a later reader knows the clean result was
  demonstrated rather than assumed.

A plotting script whose data is traceable end to end, with the traceability itself
checked, is what makes a figure safe to regenerate.

### Map every column, and watch for a near-miss field name

A tabular export is what an external reader consumes, so it must be checked against the
structured authority — not spot-checked. Two failure modes appeared in one verified case.

**Narrow coverage that passes is not coverage.** A first version verified 3 of a table's
18 columns and reported clean. Extending to every column that had a structured
counterpart raised the count from `16` to `72` checks and immediately surfaced a problem.

**A near-miss field name looks exactly like a data divergence.** The extension then
reported four disagreements — the same column, all four rows, table saying `polynomial`
and authority saying `Cartesian`. Four consistent disagreements across every row read as
a systematic divergence and would have been escalated as one. Both values were correct:
the structured record carried **two similarly named properties**
(`stretchingType = "polynomial"` and `ScalingType = "Cartesian"`), and the check had been
pointed at the wrong one. The column mapped to the first.

Practise:

- **Map every column to a named authority**, and list the columns you could not map, so
  the unchecked surface is explicit rather than implied.
- **When a whole column disagrees row after row, suspect the mapping first.** Random data
  errors do not usually affect every row identically; a systematic mismatch usually means
  the comparison itself is wrong.
- **Print the available fields before choosing one**, rather than picking the name that
  looks closest. This is the same trap as the null lookup: a plausible-looking name that
  is not the intended one.
- **Record near misses.** A false alarm that was resolved teaches the next reader where
  the ambiguity lives; deleting it hides the hazard.

### Derive the comparison tolerance from the precision the artifact carries

A fixed tight tolerance applied to a rounded column produces a false defect; loosening it
by hand until the check passes hides *which* column is coarse. Derive the tolerance from
the digits actually present.

In a verified case a column of tabulated counts and ratios was checked with a `1e-9`
relative tolerance. Two rows were flagged: the table carried a seven-figure value where the source
had a thirteen-figure value. The values agreed — the column simply carried **7 significant
figures while its neighbours carried 10 to 13**, so the tolerance demanded digits that
were never written.

Practise:

- **Compute the significant digits in the stored value** and set the tolerance from them:
  for `s` digits, accept `10^-((s-1))`. That is the honest statement of what the artifact
  can support.
- **Record the derived tolerance per cell**, so a reader can see which columns are coarse
  instead of inferring it from failures.
- **Do not tune a tolerance to make a check pass.** If a value fails, decide first whether
  the disagreement is in the data or in the comparison; only then adjust, and justify the
  adjustment from the artifact's precision rather than from a desire for green.
- **Re-derive headline constants rather than echoing them.** A summary row giving the
  maximum of a set was recomputed as the maximum of the per-gap values, which verifies
  the registered bound instead of copying it.

### Tune a whole-repository scan until its noise is suppressed, or it will be ignored

A pattern-based scan tuned on one document produces mostly false positives when applied
to a whole repository. In a verified case a structural scan of 31 markdown files gave 45
findings; inspected one by one they were **0 real leaks, 8 review items and 37 false
positives** — documented tool names, error codes, a standard-library function name, and
intentional repository URLs. Acting on those would have damaged correct documentation,
and a scanner whose false positives outnumber its true findings gets ignored.

Classify rather than delete, and encode the rules:

- **published API and error-code identifiers are surface, not private data**, so tokens
  beginning with a documented prefix belong to the noise class;
- **standard-library names** such as a hash function are not private identifiers;
- **a repository URL that is documented or cited** is intentional; only an unlisted
  remote warrants review;
- **published acceptance values in a validation reference are reference data.** They are
  correctly labelled as validating one build rather than promising portable accuracy, so
  keep them for human review instead of treating them as a leak.

Keep the strict classes genuinely strict — filesystem paths, home directories, e-mail
addresses — and defer everything else to review rather than asserting.

Then state the limit precisely: **absence of leaks by shape is not absence of leaks in
content.** A private value written as a round number, or a private fact stated in prose,
matches no pattern. Shape scanning complements reading the text; it does not replace it.

### Audit the gate CONTRACT, not only the gate result

Every check so far compared measured values against limits. None verified that the limit
is the registered one, or that the inequality points the right way. A gate can pass while
enforcing the wrong threshold or an inverted comparison, and every downstream check will
still agree with it.

Compare three independent representations:

1. the **frozen registration**, including its failure clause;
2. the **enforcement code** — which operator, against which limit;
3. the **recorded outcome**, which must follow from (2).

Two mismatches matter. Registration differing from enforcement means the registration is
not what runs. Recorded outcome differing from the enforced rule means the verdict does
not follow. Either is a contract defect regardless of whether the gate passes. Check the
**direction** explicitly: `>= 0.90` and `<= 0.90` both look like a limit, and the wrong
one inverts the verdict.

**Prefer enforcement that reads from the registration.** In a verified case the evaluator
took both the limit *and* the comparison direction out of the frozen registration, so a
gate could not disagree with its own registration by construction. Confirming that no
literal comparison bypasses it is then the whole audit, and the result was four gates
consistent with zero findings.

Two failure modes appeared while building the check, and both looked like clean results:

- **An over-broad pattern invented comparisons that do not exist** — it reported two
  extra thresholds because it matched unrelated text later on the same line. A pattern
  broad enough to find what you want is often broad enough to invent what you do not.
- **The tightened pattern then matched nothing and still reported zero findings.** Add an
  assertion that at least one enforcement was located, and let it fail loudly. Here it
  fired immediately, before the pattern was corrected — which is the assertion working,
  not obstructing.

### Registering a correction does not remove the corrected text

A supersession register records what was withdrawn. That record does not delete anything:
the withdrawn wording can survive in an artifact written before the correction and never
regenerated. Whether a reader is misled depends on where it survives, so search for it
and **classify each hit** rather than treating presence as a defect.

Three benign shapes, all observed in a verified case with 14 register entries:

- **the field holds the corrected value.** Where the correction is itself a value — a
  hash, a path, a constant — any artifact carrying the corrected value necessarily
  contains that text. Flagging it would invert its meaning: the match proves the fix was
  applied.
- **the old wording is preserved on purpose** beside the correction, so the history stays
  auditable. The register or a companion correction artifact should say so explicitly.
- **the field name declares it records the old wording** — a key containing
  `superseded`, `retained`, `original`, or `previous` is a record, not an assertion.

A genuine defect is a superseded entry whose old text appears in an **asserting** field,
with no corrected-value match, no stated retention, and a name that does not mark it as a
record. Result here: 12 hits, all classified, **zero unexplained**.

Two practical notes:

- **Look for the retention declaration beyond the entry.** It may live in a separate
  correction artifact rather than beside the text. Searching only the register entry
  produced six hits labelled unexplained that were in fact fully documented.
- **Prove the classifier can still produce the bad label.** Assert that a constructed
  violation — an asserting field, no corrected-value match, no retention, a neutral name
  — is classified as unexplained. Without that, a zero count is indistinguishable from a
  matcher that no longer fires.

Also state the limit: this searches long strings in structured artifacts. A superseded
**numeric** value, or wording in a table export or figure, needs its own check.

### Map the data flow before calling a check independent

Two checks that read the same upstream artifact cannot corroborate each other. Their
agreement is **agreement by construction**, and citing it as independent verification
overstates the evidence while every number remains correct.

Build the map mechanically: for each script, collect the artifacts it **writes** and the
artifacts it **reads**, then report every pair where one reads what another writes. In a
verified case this produced **83 dependency edges across 107 scripts**, including:

- a gate re-evaluation and a tabular recomputation that **both** read the same analysis
  artifact and the same point files, so their shared fields corroborate nothing;
- a CSV-versus-structure check that read the re-evaluation's own output, making its gate
  rows an **echo** rather than a re-derivation;
- many consumers of two central finding artifacts, each inheriting rather than
  recomputing.

Classify honestly, then cite accordingly:

- **independent** — reads the raw source or the primary data directly, so it re-derives
  rather than echoes. A raw-integral cross-check, a convention recomputed across every
  mode, an additivity check against raw regions, and a byte-level hash verification are
  independent legs.
- **agreement by construction** — inherits an upstream artifact. Still worth running, as
  a consistency check, but it must not be described as independent confirmation.

Reclassify **strength of evidence**, never the measured values. State explicitly that no
gate or value changes as a result: the point is to stop claiming more corroboration than
the structure supports, not to revise a finding.

### When the map shows everything is downstream, build a primary-evidence leg

Mapping the data flow can reveal that **every** verification reads a derived artifact, so
nothing corroborates the headline conclusion. The remedy is not to relabel the checks but
to add one that reads primary data.

In a verified case the strongest available source was a **raw integral dump** written
directly by the collection step — every quantity as a complex value per expression per
mode, with the expression list stored beside the numbers so the layout is self-describing
rather than assumed. Re-deriving the headline quantity from it reproduced the reported
values exactly.

What such a leg needs:

- **read the raw dump and nothing derived from it.** Take the complex eigenvalue and the
  selected mode index from the run record if the dump does not carry them, and **state
  that partial dependency explicitly** rather than describing the check as fully
  independent.
- **derive the headline quantity from first principles**, applying the convention already
  verified elsewhere, rather than re-reading a stored result.
- **recompute the comparison quantities too** — both the effect size and the sensitivity
  bound — so the gate outcome follows from the raw numbers rather than being inherited.
- **compare against the package and report agreement or mismatch**, with a mismatch
  meaning the package values are unconfirmed.
- **state that reproducing a gate does not pass it.** The gate failed at both levels
  before and after; a successful reproduction changes the strength of the evidence, not
  the verdict.

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

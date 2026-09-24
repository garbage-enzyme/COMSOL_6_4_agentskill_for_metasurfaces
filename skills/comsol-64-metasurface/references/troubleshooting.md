# Troubleshooting index

## Contents

- Startup and ownership
- Clientapi errors
- Geometry and mesh
- Ports and physics
- Materials and studies
- Results and validation
- Runtime and deployment

## Startup and ownership

| Symptom | Smallest safe diagnostic |
| --- | --- |
| First start call times out | Poll status; do not issue a second start while `starting=true`. |
| Parallel read-only MCP calls stall | Stop batching calls to that stdio server; wait for the original requests or deliberately restart the MCP host, then invoke capabilities/status/ownership strictly one at a time. |
| First solver-free scientific tool stalls during a cold native import | Treat this as a process-wide late-import defect, not a NumPy/SciPy-specific or transport-concurrency defect. Stop issuing calls and deliberately restart the MCP host. A server release must preload every native-backed module used in the main MCP process before event-loop dispatch, keep plotting/ML imports in bounded isolated workers, and pass a fresh-stdio matrix covering representative optimize/interpolate paths without starting COMSOL or the JVM. |
| External collision reported | Inspect exact process identity and lease; do not attach/start another client. |
| Only one client can be instantiated | Restart the Python/MCP host; do not construct another client in the same JVM. |
| Job stuck cancelling | Inspect attempt-bound phases, worker/descendants, port, lease, and coordinator identity; remain nonterminal if uncertain. |
| Visible terminal flashes | Stop broad process tests; verify the launcher/actual child PID handshake and hidden-window flags. |

## Clientapi errors

| Symptom | Likely cause/fix |
| --- | --- |
| `No matching overloads` | Direct-Model overload used on `ModelClient`; inspect Java reflection and use clientapi signatures. |
| Collection is not subscriptable | Convert tags/entities explicitly and call `.get(tag)`. |
| Feature has no `.type()` | Use tag/label/exported metadata; do not assume the direct API. |
| Operation cannot be created | Wrong study/feature type or wrong physics interface; use exact full names and dimension types. |

## Geometry and mesh

| Symptom | Likely cause/fix |
| --- | --- |
| Cannot create `fin` | It is reserved and auto-created; configure the existing finalization feature. |
| Face parameter out of range | Query `faceParamRange()` and use its midpoint. |
| Periodic source/destination incompatible | Prove translation-congruent partitions and use `FreeTri -> CopyFace -> FreeTet`. |
| CopyFace copies no target | Wrong boundary mapping/partition or missing congruence; re-probe centers, normals, adjacency, and translation. |
| Periodic port selects several faces | Cell-side classification used normals only and included internal faces; add bounding-plane coordinate tests. |
| A valid solve fails a fixed-mesh identity gate after changing wavelength or geometry | Audit which parameters drive diffraction orders, geometry, and physics-controlled meshing. Key identity by those axes, reorder the sweep accordingly, and preserve the old row under a versioned diagnostic output. |
| A later GCMMA/MMA shape step fails with NaN/Inf material coordinates | Do not classify the last accepted point as optimal or assume the iteration count is the cause. Save the accepted control/objective trajectory, replay fractions of the proposed next step in fresh forward solves, and audit relative element volume/Jacobian or frame-mapping validity. A candidate can improve the objective while already crossing into negative deformation Jacobians before a larger fraction produces NaNs. Test a smaller explicit caller-owned move limit as a causal control; do not silently shrink it, hard-code a model/host value, or use automatic method fallback. |
| `mesh.feature().remove("size")` fails with "cannot remove feature" | The default size feature is structural and cannot be removed, like the geometry `fin` feature. Keep it and add further `Size` features instead. Adding a restricted selection to the default feature itself also fails with "entity has no selection", because its type is `MeshSizeDefault` and it has no selection to restrict. Prefer creating a new `Size` feature over reconfiguring the default. |
| No vertex/coordinate accessor on the mesh stat object (COMSOL 6.4) | On the version verified here the stat object exposes only counts; vertex and connectivity arrays live on the mesh sequence. Use the mesh sequence accessors for arrays. Verify the accessor exists on your build rather than assuming a symmetric API between the mesh and its stat object. |
| Mesh-array shape or emptiness assertion fails unexpectedly (COMSOL 6.4) | On the version verified here the vertex accessor returns a component-major 3-by-N array rather than N-by-3, and the tetrahedral connectivity accessor returns fewer than two dimensions when no volume elements were generated. Inspect the actual shape and guard the empty case before transposing or indexing. |
| A domain-restricted `Size` alone generates no elements | A `Size` feature controls element size but does not create volume elements; a volume mesher covering the same domains is still required. Without it the tetrahedral element count is zero. |
| Two configurations are expected to share a mesh, but element counts differ | Do not conclude the geometry differs. Prove the sub-geometry is identical by evaluating its position and size expressions under both configurations, and test mesh reproducibility as a separate question. |
| A hash or equality check reports a match for arrays that may be absent | A `getattr`-guarded accessor probe can yield `None`, and comparing `None` to `None` reports a match. Assert the arrays are non-null and non-empty before hashing; an absent accessor is not evidence. |

### Comparing configurations with the mesh held fixed

Holding the mesh fixed is a common way to isolate one variable. Do not assume it
is attainable: verify it, and be prepared to replace the exact freeze with a
bounded discretisation estimate.

Verified behaviour to design around:

- The mesher is not guaranteed to reproduce the same elements for a sub-region
  across two different model states, even when that sub-region's geometry is
  exactly identical. In a controlled case, sub-region extents agreed to near
  machine precision, yet the sub-region mesh differed in vertex count and
  coordinates.
- Restricting the meshing features to the sub-region does not by itself restore
  reproducibility. Neither does removing boundary and copy features so that a
  bare volume mesher acts on the sub-region alone.
- Mesh reproducibility and geometry identity are therefore **independent**
  properties. Establish them with separate tests, and only the second is a
  statement about geometry.

Practical consequences:

- Do not adopt a sub-region coordinate hash as the success criterion for "the mesh
  is frozen". Prefer criteria tied to the observable being measured.
- When an exact freeze is unavailable, bound the discretisation contribution
  instead of eliminating it: refine both configurations over the same resolution
  levels, compare only within a level, and treat a between-configuration effect as
  resolved only if it exceeds the measured within-configuration mesh sensitivity.
  Otherwise report the quantity as unresolved.
- Report an unattainable freeze as a negative result, with the physical-versus-
  discretisation hypotheses stated separately. Do not present a refinement bound
  as proof that the mesh was frozen.

## Ports and physics

| Symptom | Likely cause/fix |
| --- | --- |
| `axisx/axisy` undefined | Empty/invalid `rdir1`; select a top-port edge along the intended lattice direction. |
| Missing `ewfd.Eampl*` or `TE/TM` enums appear | Wrong physics interface/namespace; create `ElectromagneticWavesFrequencyDomain`. |
| Periodic port fails near metal | Adjacent domain is not homogeneous/isotropic; use an appropriate layered boundary or redesign the port region. |
| Angle appears ineffective | Verify `alpha1_inc/alpha2_inc` on parent and both ports, evaluate them independently, and test a physical structure across signed directions. |
| `S/P` disagrees with expected field | Calibrate in a homogeneous all-air reference region; `rdir1` and incidence plane control the mapping. |

## Materials and studies

| Symptom | Likely cause/fix |
| --- | --- |
| R>1, A<0, negative lossy-domain Qh | Wrong loss sign or normalization; audit formulation-specific phasor convention and physical flux. |
| Material does not change electrostatics | Default FreeSpace feature ignores material permittivity; add ChargeConservation reading `from_mat`. |
| Dispersion frozen in sweep | Study changes frequency but not global `wl`; stage points or use an active parametric sweep. |
| Wavelength study cannot be created | Use `Wavelength` for Wave Optics, not a generic frequency-domain step. |
| Parametric list ignored | Use `plistarr`, `pname`, `punit`, and activate the sweep. |

## Results and validation

| Symptom | Likely cause/fix |
| --- | --- |
| Only last outer point is visible | Use staged one-point solves or `EvalGlobal.computeResult()`. |
| Internal absorption agrees but exceeds one | Shared normalization is internally consistent but not physical closure; evaluate declared flux planes. |
| Fine mesh changes amplitude at old peak | Re-find and bracket each mesh's own peak before comparing. |
| High Q depends on scan window | Baseline/linewidth definition is inconsistent or peak is unbracketed; persist crossings and fit residuals. |
| Headless image export creates no file | Evaluate arrays, interpolate a declared slice, and render outside COMSOL. |

## Runtime and deployment

| Symptom | Likely cause/fix |
| --- | --- |
| Sharing violation on state/lease | Use bounded retry with exact-byte revalidation; preserve foreign locks and fail closed. |
| Resume creates duplicate points | Deduplicate exact full configuration identities and load durable rows into discovery state. |
| Resource metrics are missing | Refuse when the explicit policy requires them; never fabricate telemetry. |
| Source edits do not appear live | Force non-editable reinstall, restart the MCP/CLI host, and compare deployment hashes. |
| Same version appears unchanged | Version is insufficient; compare catalog/schema/profile/build identity. |

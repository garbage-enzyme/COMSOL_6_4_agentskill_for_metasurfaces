# Native result plotting and image export

Use this module for COMSOL-native plot groups, geometry-aware field overlays,
camera control, and PNG export. Read the ownership and evidence modules too when
the plot requires loading a solved model or running a missing state.

## Contents

- Prefer model topology over reconstructed polygons
- Surface selection uses a plot-attribute child
- Use a shared symmetric field range
- Export Image3D with the plot-group overload
- Views and camera coordinates
- GUI and standalone boundaries

## Prefer model topology over reconstructed polygons

Use COMSOL result features when a field must lie on exact physical boundaries.
Do not infer visible faces from a drawing, a point cloud, or construction order.
For every geometry state:

1. Probe `getUpDown()`, `faceParamRange()`, `faceX()`, and `faceNormal()`.
2. Classify boundaries by adjacent domains, center, and normal.
3. Require the expected count and orientation before plotting.
4. Re-probe mirrored, signed, rebuilt, or topology-changing geometry. Boundary
   numbers can change even when the intended physical category does not.

Treat an internal material-material interface, an exposed material-air face,
and a numerical cut plane as different evidence. A plot that looks plausible on
the wrong category is not a valid field rendering.

## Surface selection uses a plot-attribute child

A 3D `Surface` or `Slice` result feature may not expose a direct entity
selection. Create a `Selection` plot-attribute child instead:

```python
results = jm.result()
group = results.create("pg_field", "PlotGroup3D")
group.set("data", "dset1")
group.set("titletype", "none")
group.set("showlegends", False)
group.set("edges", "off")

field = group.feature().create("surf_field", "Surface")
field.set("expr", JArray(JString)(["real(ewfd.Ez)"]))
field_selection = field.feature().create("sel1", "Selection")
field_selection.selection().set(JArray(JInt)(field_boundary_ids))
```

Calling `field.selection()` can raise `FlException: entity has no selection`.
The child-node pattern matches the Results tree and is portable across GUI Java
Shell and `ModelClient` use.

For a gray geometry context, add a second `Surface` with expression `0`, uniform
custom gray coloring, and a selection containing the remaining solid
boundaries. Exclude the field boundaries from this gray complement so the two
surface features do not compete for the same pixels.

Set `PlotGroup3D.edges="off"` when the solution dataset draws an unwanted air
box. This removes dataset edges; it is independent of the selected surface
features.

## Use a shared symmetric field range

For signed real fields, derive a bound on the exact plotted boundaries with a
`MaxSurface` numerical feature and plot every compared state with one symmetric
range:

```python
maximum = numerical.create("max_field", "MaxSurface")
maximum.set("expr", JArray(JString)(["abs(real(ewfd.Ez))"]))
maximum.selection().set(JArray(JInt)(field_boundary_ids))
raw = maximum.computeResult()
limit = margin * float(list(raw[0][0])[0])

field.set("rangecoloractive", "on")
field.set("rangecolormin", f"{-limit:.17g}")
field.set("rangecolormax", f"{limit:.17g}")
```

Use the same declared limit, phase convention, expression, dataset, and view for
every compared state. Check every later state's boundary maximum against the
limit and fail rather than clip. A suppressed-state all-node maximum is a valid
conservative no-clipping bound when its exact boundary maximum is unavailable.

## Export Image3D with the plot-group overload

Create the image export with the three-argument overload. A two-argument
`Image3D` plus a guessed `source` property can run without creating a file.

```python
group.run()
image = results.export().create("img_field", "pg_field", "Image3D")
image.set("plotgroup", "pg_field")
image.set("imagetype", "png")
image.set("pngfilename", str(output_png))
image.set("size", "manualweb")
image.set("width", "1200")
image.set("height", "900")
image.set("antialias", "on")
image.run()
```

Require that the PNG exists, is nonblank, has a plausible size, and receives an
image-capable review. Hash the PNG and bind it to the source, dataset, expression,
selection categories, range, and camera/view identity.

## Views and camera coordinates

Let an automatic result view render once before changing its camera. Do not
blindly bind a new result group to an uninitialized geometry view; headless
export can become blank.

COMSOL camera `position`, `target`, and `up` use scene coordinates, which can be
normalized values rather than model length units. Read the initialized camera
with `getDouble(property, index)` before changing it. Preserve `projection`,
`zoomanglefull`, `viewscaletype`, and `autoupdate` with the vectors. If the
automatic headless view already matches the required orientation, keep it; this
is more stable than reconstructing a camera from model coordinates.

## GUI and standalone boundaries

The Results-tree techniques in this module are execution-surface independent.
They apply in Desktop Java Shell, direct `ModelClient` code, and an MCP
attached/shared interactive session when live capability discovery exposes a
generic ClientAPI, plotting, or image-export operation that can express the
exact call. In an interactive MCP session, keep the same topology probes,
selection assertions, shared ranges, view initialization, immutable-source
rules, and artifact checks; serialize every MCP call and preserve the shared
Desktop/Server ownership contract. User-guided GUI inspection can establish
which physical faces are intended, but the final artifact must still record and
re-probe their adjacency, centers, normals, and current entity IDs.

Prefer an MCP plotting/export tool when the active profile exposes one. If the
healthy MCP profile has no plot-group or image-export surface, treat that exact
operation as unavailable rather than bypassing an available solver operation.
Release the MCP session, require a fresh collision-free inventory, and use one
dedicated standalone process only for the bounded native plotting task. Do not
overlap it with Desktop or another MPh client.

COMSOL Desktop Java Shell obeys its File system access security preference. An
export outside the allowed roots can raise `AccessControlException`. Do not
weaken the security preference automatically. Export to an already allowed
location or close Desktop without saving and perform the bounded standalone
export after ownership is clean.

Treat the source MPH as immutable: hash it before and after, never save the
temporary Results tree, remove the model from the client, and verify process and
lease cleanup. A regenerable plot does not justify retaining another MPH. Keep
stdout receipts concise; large diagnostic property dumps can block a piped
worker after its scientific artifacts are already complete.

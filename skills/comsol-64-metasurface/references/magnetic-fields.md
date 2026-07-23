# AC/DC magnetic-field workflows

## Scope

Use this reference for an AC/DC Magnetic Fields or Induction Currents baseline,
Coil features, or a one-point magnetic-field smoke. Treat a supplied `.mph` as
an interface scaffold until its geometry, materials, winding convention, and
study setup have been independently inspected.

## Baseline and ClientAPI checks

- A physics tag such as `mf` is not a module identity. Inspect its label,
  required default features, and the available study feature types before
  mutating a derived copy.
- When a required AC/DC interface cannot be created reliably from scratch, load
  a compatible baseline, hash it, and only save a distinct derived model.
- Java tag arrays contain `java.lang.String` values. Convert before Python text
  logic: `name = str(tag)`; use the original tag for ClientAPI collection calls.
- Do not assume a generic study type is valid for every baseline. Some
  compatible AC/DC baselines accept `Frequency`, while others accept
  `FrequencyDomain`. Export a trusted model as Java or perform a bounded
  derived-copy probe before selecting the feature type.

## Standalone lifecycle

```python
client = mph.Client(version="6.4")
model = client.load(str(source))
try:
    jm = model.java
    # Mutate only the derived in-memory model.
    jm.save(str(output.resolve()))
finally:
    client.remove(model)
```

- `model.remove()` removes a model node and is not whole-model cleanup.
- Use `client.remove(model)` for a loaded standalone model. Do not call
  `client.disconnect()` merely because a standalone client is local; it is for
  a connected server session and can fail when no server connection exists.
- Start a fresh Python process for a later standalone client if JVM state is
  uncertain.

## One-point smoke sequence

1. Run solver-free ownership, process, memory, source-path, and output-path
   preflight.
2. Confirm no solver lease, active durable job, or external COMSOL/MPh process.
3. Build one mesh and solve one declared frequency point on a unique output.
4. Preserve the raw field expression, source/output SHA-256 values, output size,
   and post-run process-cleanup result.
5. Re-hash the immutable source after cleanup.

A finite field value or agreement with an inherited numerical note is only a
smoke result. Physical acceptance additionally needs declared current and
winding convention, material evidence, geometry/mesh adequacy, and an
independent analytical or benchmark comparison appropriate to the claim.

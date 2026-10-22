# Standalone fallback for an unavailable MCP operation

Read this reference only when the required COMSOL operation is unavailable
through the active MCP surface. Prefer the MCP server whenever it can accept the
exact serialized operation. It supplies ownership, containment, durable-state,
and evidence controls that direct MPh does not recreate.

## Gate

Use direct MPh only after one of these terminal conditions:

- no COMSOL MCP client or server is configured on the host.
- the server executable/import is unavailable.
- a serialized startup attempt completed with a terminal failure.
- live capability discovery proves that the required bounded operation, such as
  native plot-group/image export, is absent from the active profile and no safe
  generic MCP operation can express it.

Do not use this fallback to bypass a profile, containment rule, input bound, or
MCP error. An absent operation authorizes only that bounded fallback task, not a
solver or mutation already exposed by MCP. Do not use it while `comsol_start` is
in flight, after a timeout whose original request may still run, or while a
healthy MCP session is connected. Disconnect and verify lease release first.

Before creating a direct client, perform a fresh process inventory and refuse if
any COMSOL, Java, MPh, solver-worker, or uncertain lease owner exists. Do not
start a second client to recover a failed direct-client process. Terminate that
Python process and begin again only after a fresh inventory.

## Install MPh

COMSOL Multiphysics must already be installed and licensed locally. Create a
non-editable, ASCII-path environment. Never install a fallback into the project
source tree:

```powershell
$env:COMSOL_STANDALONE_ENV = Join-Path $env:LOCALAPPDATA 'comsol-standalone'
py -3.14 -m venv $env:COMSOL_STANDALONE_ENV
& (Join-Path $env:COMSOL_STANDALONE_ENV 'Scripts\python.exe') -m pip install --upgrade pip
& (Join-Path $env:COMSOL_STANDALONE_ENV 'Scripts\python.exe') -m pip install "mph==1.3.1" "jpype1==1.7.1"
```

When discovery cannot find the local COMSOL runtime, set process-local Java
paths before importing `mph`, using the Java runtime shipped with the installed
COMSOL version:

```powershell
$env:COMSOL_ROOT = '<absolute path to the installed COMSOL Multiphysics root>'
$env:JAVA_HOME = Join-Path $env:COMSOL_ROOT 'java\win64\jre'
$env:JDK_HOME = $env:JAVA_HOME
```

These paths are examples, not portable defaults. Verify the installed COMSOL
root and requested version first. Direct MPh is not a license bypass.

## Minimal isolated run

Use one source model, one declared point, and one unique output path. Hash the
source before and after. Save through the Java ClientAPI so Unicode destinations
remain reliable.

S01 safety rules for this example:

- Reject when source and output resolve to the same file identity
  (`os.path.samefile`), not only when the path strings differ.
- Refuse to overwrite an existing output unless the caller explicitly allows it
  after hashing the prior file.
- Use an exclusive temporary output and publish atomically. Never save directly
  onto the source path.
- Keep runtime roots on ASCII paths. On a Chinese-username home, do not fall
  back to `%LOCALAPPDATA%` if it is non-ASCII. Declare an ASCII root instead.
- Always run load/save/remove/source-check. A cleanup error must not skip the
  immutable-source verification.

```python
from hashlib import sha256
import os
from pathlib import Path

import mph


def file_hash(path: Path) -> str:
    digest = sha256()
    with path.open("rb") as handle:
        for block in iter(lambda: handle.read(1024 * 1024), b""):
            digest.update(block)
    return digest.hexdigest()


source = Path(os.environ["COMSOL_SOURCE_MODEL"]).expanduser().resolve()
output = Path(os.environ["COMSOL_OUTPUT_MODEL"]).expanduser().resolve()
runtime_root = Path(os.environ.get("COMSOL_ASCII_RUNTIME", "")).expanduser()
if not str(runtime_root).isascii():
    raise RuntimeError("COMSOL_ASCII_RUNTIME must be an ASCII path")
if not source.is_file():
    raise RuntimeError("source model is missing")
if output.exists():
    try:
        if os.path.samefile(source, output):
            raise RuntimeError("output aliases the immutable source")
    except FileNotFoundError:
        pass
    if os.environ.get("COMSOL_ALLOW_OVERWRITE") != "1":
        raise RuntimeError("output already exists; set COMSOL_ALLOW_OVERWRITE=1 after hashing")
tmp_output = output.with_name(output.name + ".tmp-partial")
if tmp_output.exists():
    raise RuntimeError("temporary output already exists")

source_hash = file_hash(source)
client = None
model = None
try:
    client = mph.Client(version="6.4")
    model = client.load(str(source))
    jm = model.java
    # Apply only a declared, bounded derived mutation and one study point.
    jm.save(str(tmp_output))
    if not tmp_output.is_file() or tmp_output.stat().st_size <= 0:
        raise RuntimeError("temporary output was not written")
    os.replace(tmp_output, output)
finally:
    if model is not None and client is not None:
        try:
            client.remove(model)
        except Exception:
            pass
    if tmp_output.exists():
        try:
            tmp_output.unlink()
        except OSError:
            pass

if file_hash(source) != source_hash:
    raise RuntimeError("immutable source changed")
```

- Use `client.remove(model)`, not `model.remove()`, to release a loaded model.
- Do not call `client.disconnect()` for a standalone client that never connected
  to a COMSOL Server.
- Use `str(tag)` before applying Python string operations to Java tag values.
- Probe study feature names on a derived copy or exported trusted model. Do not
  assume every baseline accepts the same study type.

## Manual controls and evidence

Record the preflight process inventory, source/output absolute paths, source and
output SHA-256 values, output size, exact COMSOL/MPh/JPype/Python versions,
declared inputs, raw evaluated values, and post-run process absence. Persist
each point before starting another.

Direct MPh lacks MCP's solver lease, path containment, durable job/cancellation,
profile discovery, and evidence-integrity controls. Apply those checks manually
or label the result as a limited standalone smoke. A successful native call or
finite number is not physical validation. Use declared material, geometry,
mesh, boundary, convergence, and independent-reference evidence for the claim.

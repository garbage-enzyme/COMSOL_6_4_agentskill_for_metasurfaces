# Host resource exhaustion and workflow recovery

## Contents

- Recognize the incident
- Freeze new work
- Preserve solver ownership and durable progress
- Check host recovery
- Restore MCP access
- Resume safely
- Reduce recurrence risk
- Out-of-core workspace memory behavior

## Recognize the incident

A heavy COMSOL solve can exhaust RAM, system commit, page file, handles,
process capacity, or temporary storage by itself. Other heavy work can also
push an admitted solve over the limit. Common Windows signals include
`WinError 1450`, `WinError 1455`, memory or process-creation failures, a broken
stdio pipe, or an MCP host that disappears while a solver may still exist.

Treat this first as a host incident. Do not infer that the model, physics, or
last solved point is invalid, and do not infer that COMSOL stopped merely
because the MCP transport vanished.

## Freeze new work

1. Stop issuing requests to the affected COMSOL MCP server. Do not retry a
   timed-out request while it may still be running.
2. Start no new solve, client, sweep, render, or other heavy workload.
3. Preserve journals, completed point artifacts, logs, checkpoints, and the
   latest status file. The append-only journal remains completion authority.
4. Do not terminate a user-owned or uncertain COMSOL, Java, Server, Desktop,
   launcher, or MPh process.

## Preserve solver ownership and durable progress

Inventory the solver lease and the exact COMSOL, Java, MPh Python, launcher,
worker, and descendant identities. If inventory is incomplete or misses its
deadline, report uncertainty and fail closed.

If the exact owned durable launcher supports pause and the caller authorizes
it, request pause and wait for the next flushed point boundary. Treat pause as
acknowledged only when durable state and owned-process evidence agree. Never
claim the interrupted point complete unless its full row and bound artifacts
were flushed and verified.

If the solver is user-owned, unknown, or independently launched, leave it
running. Recovery work must adapt around it rather than taking ownership.

## Check host recovery

Check physical and available RAM, system commit or virtual-memory headroom,
page-file use, free disk, process count, and relevant handle counts when
available. Verify that no owned helper or abandoned worker remains. Low CPU
usage alone is not proof of recovery.

If the solve itself approached the host limit, do not simply resume with the
same resource request. Reduce admitted cores or memory mode, split the campaign,
use smaller durable stages, or move the work to a larger host.

## Restore MCP access

Restore the MCP transport without starting another COMSOL client:

1. Ensure no earlier stdio request remains in flight.
2. Restart only the owning MCP client/session so it creates one fresh stdio
   host. Do not leave a manually spawned duplicate host attached to the same
   runtime.
3. Call `capabilities`, `comsol_status`, and `solver_status` strictly one at a
   time. The goal is truthful transport and ownership state, not solver startup.
4. If an external solver still exists, preserve it and follow the declared
   attach or durable-resume contract. Do not call `comsol_start` merely to test
   connectivity.

## Resume safely

Resume only when source, configuration, driver, journal, attempt, lease, and
solver-owner identities match. Replay completed rows as skipped. If the
interrupted point or owner is uncertain, remain nonterminal and require operator
review.

Before admitting another point, apply the caller-declared resource policy to
current headroom plus the observed peak and cleanup margin. For materially
changing geometry, smoke both expected high-resource endpoints before a long
unattended run.

## Reduce recurrence risk

- Treat a heavy COMSOL solve as capable of exhausting the host alone.
- Avoid overlapping it with another solver, high-order RCWA, large rendering,
  or other resource-intensive work.
- Use one-point durable execution, bounded core counts, explicit memory policy,
  and flush plus `fsync` before the next point.
- Recalculate runtime and peak-resource estimates from measured points rather
  than carrying assumptions from a different mesh or solver backend.
- Archive a path-redacted recovery receipt with the error, resource snapshot,
  last durable point, ownership evidence, cleanup, and resume decision.

## Out-of-core workspace memory behavior

A direct solver with the out-of-core property set to `on` does not stay small.
Its workspace allocates elastically until nearly the whole system commit limit
is used. Measured peaks reached about 98.4 percent of commit from start
baselines between 0.66 and 0.84. The allocation follows free headroom, not a
fixed size. A whole-host commit ceiling below that level trips at the solve
even on an otherwise lightly loaded host.

Setting the property to `off` is memory-bounded. A small periodic wave-optics
model with a deformation stage solved with a measured peak near 0.80
whole-host fraction instead of 0.98. The numeric result matched the
out-of-core run to 1.8e-11. Setting the property to `auto` also bounds memory,
but its spill machinery can invalidate post-solve object references. A COMSOL
error `variable points to an object that no longer exists in the model`
appeared during post-solve evaluation under `auto` and did not appear under
`off`.

When a caller enforces a whole-host commit ceiling, prefer `off` for small
models. Record the out-of-core setting with the run evidence. A ceiling trip
under `on` does not indicate a defect in the model, the mesh, or the worker.

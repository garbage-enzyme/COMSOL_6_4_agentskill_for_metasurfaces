# Runtime reliability development

## Contents

- MCP transport and solver ownership
- Durable state authority
- Process identity and cancellation
- Atomic Windows I/O
- Admission races and status inventories
- Resource exhaustion
- Recovery and regression evidence

## MCP transport and solver ownership

Serialize every request to one stdio server, including capabilities and status.
Server-side operation classes do not make concurrent stdio requests safe. After
an interrupted wait, assume the original request may still run until a known
terminal result or deliberate host restart.

Keep one solver owner. Before client creation or lifecycle mutation, require a
fresh complete process inventory plus lease validation. Read-only status may use
a labeled recent cache; mutation may not.

Use two bounded clean snapshots when a newly starting external process can race
with admission. A single clean inventory can become stale before a worker starts.

## Durable state authority

Bind each job attempt to immutable source, normalized configuration, driver,
runtime, and attempt identities. Persist:

- immutable specification and fingerprints;
- atomic projected state;
- append-only event/result/resource journals;
- attempt-bound control requests;
- checkpoints, bounded logs, and cleanup evidence.

The append-only journal is completion authority. Mutable status is a projection
and can lag after a sharing or publication failure. Resume only exact matching
rows and replay completed points as skipped.

Flush and `fsync` each durable append before callbacks, progress publication, or
the next point. Test the exact descriptor and ordering, not merely that an
`fsync` call occurred somewhere.

## Process identity and cancellation

Record PID, creation time, command signature, parent identity, and descendants.
Do not terminate by process name or substring. Treat PID reuse, coordinator
restart, missing creation time, and incomplete inventory as uncertainty.

Cancellation remains nonterminal until the owned worker, descendants, ports,
leases, state transitions, and cleanup evidence agree. Bind control requests to
the attempt so a stale request cannot affect a replacement worker.

Retain process-owner objects until children exit. Dropping `Popen` or pipe
owners while durable children still run can produce warnings, blocked output,
or interpreter-shutdown stalls. Use hidden windows for noninteractive helpers.

## Atomic Windows I/O

Write through a unique temporary file, flush, `fsync`, replace atomically, and
remove residue on failure. Retry only recognized transient Windows sharing
errors within a bounded deadline. Revalidate exact bytes after ambiguous
replacement outcomes.

Test delayed sharing denial beyond the former retry window, deadline cleanup,
foreign-lock preservation, multiple writers, and exact replay. Do not convert a
bounded sharing retry into an unbounded loop.

## Admission races and status inventories

Measure inventory deadlines from one absolute request start. Starting the clock
after a worker thread or subprocess launches can allow a late result to appear
fresh. A read-only cache refresh may continue in the background, but incomplete
or expired results cannot acquire a lease, start a worker, recover an orphan, or
claim cleanup.

Cold Windows process metadata can be much slower than warm scans. Use targeted
identity probes for a declared lease owner when possible, but do not replace the
full external-collision inventory with a targeted result.

## Resource exhaustion

A heavy COMSOL solve can trigger `WinError 1450`, commitment-limit errors,
handle exhaustion, or process-creation failure by itself. Parallel tests,
coverage, builds, plotting, or another solver can amplify the risk but are not
required for this classification.

Never overlap a heavy solve with a full quality or release gate. Before a local
parallel suite, verify that no COMSOL/Java/MPh solver owner is active and that
host memory, commit, page-file, process, handle, and disk headroom are normal.

If a gate exhausts resources and the MCP host disappears:

1. stop new tests and MCP calls;
2. preserve solver journals and user-owned launchers;
3. inventory whether COMSOL or Java survived independently;
4. clean only exact owned test descendants;
5. wait for commit, memory, handles, processes, and disk to recover;
6. validate the installed MCP command outside the checkout;
7. restart one fresh stdio host without starting COMSOL;
8. query capabilities and ownership status serially.

One resource incident can expose a second deployment defect. For example, a
previous host may remain alive after a package migration while its configured
restart command still points to a repository-only module. Diagnose trigger and
direct restart failure separately.

## Recovery and regression evidence

Archive the exact error, stage, process tree, resource snapshot, last durable
point, journal tail, cleanup, configured entry point, installed build identity,
and restart result. Redact private paths before publication.

Reproduce deterministic assertions after host recovery. Do not label them
resource failures merely because the same run also exhausted resources. For an
intermittent lifecycle fix, require repeated clean runs and exact-SHA CI, not one
successful retry or a larger timeout.

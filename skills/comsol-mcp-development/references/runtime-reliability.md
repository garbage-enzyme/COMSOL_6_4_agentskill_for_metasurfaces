# Runtime reliability development

## Contents

- MCP transport and solver ownership
- Durable state authority
- External manifests and staged replay
- Process identity and cancellation
- Atomic Windows I/O
- Admission races and status inventories
- Resource exhaustion
- Typed native-condition fixture prerequisites
- Recovery and regression evidence

## MCP transport and solver ownership

Serialize every request to one stdio server, including capabilities and status.
Server-side operation classes do not make concurrent stdio requests safe. After
an interrupted wait, assume the original request may still run until a known
terminal result or deliberate host restart.

Keep one solver owner. Before client creation or lifecycle mutation, require a
fresh complete process inventory plus lease validation. Read-only status may use
a labeled recent cache. Mutation may not.

Classify collision candidates by exact process role. The installed
`comsol-mcp.exe` console entry point is a solver-free stdio host until it owns an
actual COMSOL/MPh/Java solver child, so a broad `^comsol.*` rule is incorrect.
Exclude only that exact host executable, continue to reject real COMSOL and MPh
server executables plus COMSOL/MPh Java runtimes, and add deterministic fixtures
for both accepted and rejected names. The host exclusion never overrides an
active solver lease or a detected descendant.

Use two bounded clean snapshots when a newly starting external process can race
with admission. A single clean inventory can become stale before a worker starts.

## Durable state authority

Bind each job attempt to immutable source, normalized configuration, driver,
runtime, and attempt identities. Persist:

- immutable specification and fingerprints.
- atomic projected state.
- append-only event/result/resource journals.
- attempt-bound control requests.
- checkpoints, bounded logs, and cleanup evidence.

The append-only journal is completion authority. Mutable status is a projection
and can lag after a sharing or publication failure. Resume only exact matching
rows and replay completed points as skipped.

Recovery tests must cover a partial final record, exact truncation to the last
complete record, and the next hash-chained append. Assert recovered rows and
chain identity. Do not couple the contract to the private lifetime or filename
of the synchronization lock.

Flush and `fsync` each durable append before callbacks, progress publication, or
the next point. Test the exact descriptor and ordering, not merely that an
`fsync` call occurred somewhere.

## External manifests and staged replay

Keep public job submission compact when a complete typed specification would
make core discovery unbounded. Accept only a contained ASCII JSON path plus its
exact SHA-256, then expand the file through one strict versioned normalizer.
Reject unknown fields, changed bytes, path escape, excessive size, and an
incompatible job discriminator before ownership or client construction.

Pin and re-hash the submission file, immutable source, normalized manifest,
producer, and worker identities before client startup and after execution. The
external file is transport, not an escape hatch: the expanded manifest must
remain a complete closed contract with the same schema/runtime valid set.

For a multiphysics replay, persist one hash-chained row per bounded stage. Resume
only verified complete stages and append a missing row for orphaned complete
evidence. Never repeat an accepted stage merely because projected status lagged.
Bind every result read to an explicit study and dataset so a default solution
cannot substitute for the intended stage.

Keep a current-model save distinct from a checkpoint Save Copy. Verify the
working model path after each operation, preserve caller-owned sources, and
rehash the source after cleanup. Keep optional native imports and COMSOL product
checks behind solver-free structural validation so catalog discovery and job
preview cannot load native libraries or launch processes.

## Process identity and cancellation

Record PID, creation time, command signature, parent identity, and descendants.
Do not terminate by process name or substring. Treat PID reuse, coordinator
restart, missing creation time, and incomplete inventory as uncertainty.

Cancellation remains nonterminal until the owned worker, descendants, ports,
leases, state transitions, and cleanup evidence agree. Bind control requests to
the attempt so a stale request cannot affect a replacement worker.

For child-process acceptance, isolate output per case, retain the exact process
object, drain redirected streams, apply a bounded wait, and terminate only that
identity on timeout. A parent test that requires a readiness marker, proves the
child remains alive, and force-terminates it already covers early-return risk.
Do not add a second assertion that cannot observe a distinct failure.

Retain process-owner objects until children exit. Dropping `Popen` or pipe
owners while durable children still run can produce warnings, blocked output,
or interpreter-shutdown stalls. Use hidden windows for noninteractive helpers.

Windows CPython venv launchers can create another Python process.
Popen.pid can identify the launcher while os.getpid() identifies the executing worker.
Bind durable worker records and watchdog targets to the executing process identity.
For a known redirector, require one direct child with the exact launched command.
Check creation times and recheck the launcher identity after child enumeration.
Refuse missing, ambiguous, reused or mismatched identities within the existing deadline.
Include a real Windows venv path with spaces in process-only acceptance.
Check completion, cancellation and resume with COMSOL closed.

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

1. stop new tests and MCP calls.
2. preserve solver journals and user-owned launchers.
3. inventory whether COMSOL or Java survived independently.
4. clean only exact owned test descendants.
5. wait for commit, memory, handles, processes, and disk to recover.
6. validate the installed MCP command outside the checkout.
7. restart one fresh stdio host without starting COMSOL.
8. query capabilities and ownership status serially.

One resource incident can expose a second deployment defect. For example, a
previous host may remain alive after a package migration while its configured
restart command still points to a repository-only module. Diagnose trigger and
direct restart failure separately.

## Typed native-condition fixture prerequisites

A typed native-condition forward job assumes the frozen source model carries
standard auto-generated solver sequences. The worker resolves
`solution_tag -> stationary_solver_tag -> linear_solver_tag`, such as
`sol1 -> s1 -> d1`, before it regenerates sequences for the deformation stage.
A source without these objects fails with a COMSOL `Unknown solution` or
`Unknown SolverFeature` error. A source rebuilt or restored by hand must run
`createAutoSequences` and read back the exact declared structure first.

A restored source can come from a surviving derived clone. Remove only the
nodes the adapter and collector create: the deformation physics interface, the
deformation study step, the control parameters, and, depending on the clone
state, the solver sequences. Then rebuild standard sequences and verify the
fixture identities by live readback: control geometry, material domain map,
and study shape. Keep the original clone unchanged and re-verify its hash.

The fixture `condition_controls` freeze an `out_of_core_value`. The value `on`
drives the out-of-core workspace to allocate elastically to about 98.4 percent
of whole-host commit and trips any whole-host ceiling at the shape solve. The
value `off` is memory-bounded and numerically equivalent to `on` within 1.8e-11
on a small periodic model. The value `auto` bounds memory but its spill
machinery can break post-solve object references. Freeze `off` for fixtures
that run under whole-host commit ceilings, and treat a ceiling trip under `on`
as a policy conflict, not a worker defect.

## Recovery and regression evidence

Archive the exact error, stage, process tree, resource snapshot, last durable
point, journal tail, cleanup, configured entry point, installed build identity,
and restart result. Redact private paths before publication.

Reproduce deterministic assertions after host recovery. Do not label them
resource failures merely because the same run also exhausted resources. For an
intermittent lifecycle fix, require repeated clean runs and exact-SHA CI, not one
successful retry or a larger timeout.

## Native startup timeout and lease-release failure probes

Use an isolated runtime for native startup fault probes.
Record actual native constructor entry and return times.
A readiness handshake before entry can coordinate a fault without delaying the native constructor.
When a test uses a shorter watchdog, retain the production value and label the test override.
Prove that the timeout occurs between actual constructor entry and return.
Do not describe this result as a naturally occurring production-duration hang.

On Windows, a real handle without delete sharing can deny lease deletion after successful native client retirement.
Distinguish this lease-release failure from a native client.clear exception.
Verify retained cleanup_pending state, retained lease ownership, durable terminal state and refusal of a second start.
Verify that the native constructor executes only once.
Close the fault handle during final cleanup.
Prove exact child exit before stale recovery.
Verify fresh process, lease and job inventories after recovery.
Keep deterministic client-retirement failure controls separate from this native lease-release evidence.

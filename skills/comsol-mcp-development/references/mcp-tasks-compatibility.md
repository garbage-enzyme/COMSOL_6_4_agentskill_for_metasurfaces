# MCP Tasks and legacy clients

Use this reference when changing the MCP SDK or exposing a nonblocking
simulation start through native Tasks.

- Verify the installed SDK version, negotiated date-based core protocol, and
  Tasks extension generation independently. An SDK major upgrade does not
  prove extension support. Check public SDK hooks and actual wire messages.
  Avoid monkey-patching private dispatch.
- Advertise the extension only when implemented and require the client's
  per-request capability opt-in before returning a task-shaped result.
  Test absence, wrong generation, and clients that recognize no Tasks at all.
- Persist the task-to-existing-durable-job mapping before acknowledgement.
  Idempotent retries, disconnect/reconnect, restart, expiry and authorization
  must not create another solver job or expose another owner's result.
- A prompt acknowledgement is distinct from solve completion. Cancellation
  acknowledgement is distinct from cleanup: preserve cleanup-pending status
  until exact owned process, port, lease and job observations settle.
- Keep ordinary profile-scoped job submit/status/tail/cancel/resume for older
  clients. Test named versions through real stdio and installed-wheel probes.
  Do not claim support for every client from SDK major alone.
- Retain one scheduler and one serialized COMSOL owner. Protocol-level
  concurrency never authorizes simultaneous Java calls or a second solver.

## Client evidence

Distinguish native-client calls, an SDK test harness, and unit tests.
Record the actual handshake and per-request capabilities. Do not infer them from the client application version.
A Python script written by a client agent is not proof of that client's native Tasks support.
Record a connection failure as a failure or blocker, not as a compatibility pass.
Keep untested, unsupported, and failed cases separate.
For nonblocking acceptance, record the submit response before the job reaches its terminal state.
Check that the control channel remains usable while the job runs.
For cancellation acceptance, observe a running job before cancellation and collect the final cleanup evidence.

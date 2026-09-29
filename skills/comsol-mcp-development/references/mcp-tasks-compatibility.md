# MCP Tasks and legacy clients

Use this reference when changing the MCP SDK or exposing a nonblocking
simulation start through native Tasks.

- Verify the installed SDK version, negotiated date-based core protocol, and
  Tasks extension generation independently. An SDK major upgrade does not
  prove extension support. Check public SDK hooks and actual wire messages;
  avoid monkey-patching private dispatch.
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
  clients. Test named versions through real stdio and installed-wheel probes;
  do not claim support for every client from SDK major alone.
- Retain one scheduler and one serialized COMSOL owner. Protocol-level
  concurrency never authorizes simultaneous Java calls or a second solver.

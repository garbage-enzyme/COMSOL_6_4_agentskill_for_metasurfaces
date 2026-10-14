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

Read the Tasks wire generation before selecting a result method.
In stable SEP-2663, tasks/get carries the terminal result and tasks/result is removed.
After completed native work, a client-harness error must not trigger automatic resubmission.
Verify the original durable rows, task mapping and terminal result through a fresh client.

## Client evidence

Distinguish native-client calls, an SDK test harness, and unit tests.
Record the actual handshake and per-request capabilities. Do not infer them from the client application version.
A Python script written by a client agent is not proof of that client's native Tasks support.
Record a connection failure as a failure or blocker, not as a compatibility pass.
Keep untested, unsupported, and failed cases separate.
For nonblocking acceptance, record the submit response before the job reaches its terminal state.
Check that the control channel remains usable while the job runs.
For cancellation acceptance, observe a running job before cancellation and collect the final cleanup evidence.

## Stock DSH client capability boundary

The stock dsh-mcp-client in DSH0.2.1-alpha.2 uses ordinary tools/call dispatch.
Under2025-11-25, it refuses tools with execution.taskSupport=required before wire dispatch.
Under2026-07-28, its observed requests declare empty client capabilities and do not opt into Tasks.
These results establish ordinary fallback and explicit refusal for that pinned client.
They do not establish native Tasks support or apply to every DSH version.

Use a fixture that follows the negotiated protocol.
The2026 tool schema removes the legacy execution field.
The2026 tools/list response requires ttlMs and cacheScope.
Record each transport generation because automatic version negotiation can reopen stdio.
Keep protocol-fixture failures separate from native client failures.

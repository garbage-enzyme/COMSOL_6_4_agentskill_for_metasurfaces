# DeepSeek Harness bridge

## Current version boundary

The companion COMSOL MCP alpha7.2 release line is Python package `0.7.2`.
Its optional repository component is `@local/dsh-comsol-bridge` `0.1.0`,
requires Node.js `>=20`, and has no npm dependencies. Production may remain on
`0.7.1`; a bridge push or source release is not production deployment.

The bridge is repository-only and excluded from the Python wheel and sdist. It
must not become a Python server feature, public server schema, shared
`settings.json` field, COMSOL Settings GUI Boolean, solver-owner path, or
COMSOL compatibility-matrix entry.

## Contract

The DSH Cordis plugin owns one MCP stdio connection and serializes tool calls,
status/tail polling, and cancellation through one single-flight queue. Do not
configure a second COMSOL entry through `dsh-mcp-client` in the same DSH
session.

The Python server remains authoritative for durable journals, leases, process
identity, source immutability, cleanup, resume, and scientific evidence. A DSH
completion notice is not an independent solve receipt. `config.enabled` is a
DSH plugin switch, not a server or Settings GUI setting.

## Required synchronization

For a bridge change, inspect and update the bridge implementation, tests, smoke
script, `package.json`, and `README.md`, plus the companion server's user docs
and layout inventory when the user-visible contract changes. Keep the detailed
contract in the companion server's `dsh_bridge/DEEPSEEK_COMPATIBILITY.md`.

Retain the adapted upstream acknowledgement for commit
`99172f8f43c6753c2442c406cd5c6055ea8c5bef` in public documentation.

## Verification

Run from the companion server repository root:

```powershell
npm test --prefix dsh_bridge
npm run smoke --prefix dsh_bridge
```

The accepted solver-free bridge baseline is 37 Node tests plus fake-server
smoke. Separately verify live DSH discovery, capabilities, solver ownership,
durable job state, and settings-change host restart. The user has independently
accepted the settings-change path; this does not establish production
cancellation or licensed COMSOL solve acceptance.

Never promote fake-server behavior into production or licensed evidence. A lost
bridge process does not prove the durable job stopped: reconnect, inspect
`job_status`, and use the server's exact `job_resume` contract. Cancellation is
terminal only after server terminal state and cleanup/lease evidence are
available.

# DeepSeek Harness bridge development

## Current version boundary

COMSOL MCP alpha7.2 uses Python package version `0.7.2`. The optional repository
component `dsh_bridge/` is `@local/dsh-comsol-bridge` version `0.1.0`, requires
Node.js 20 or newer, and has no npm dependencies. Production installation may
remain on `0.7.1`; do not describe a source release or bridge push as production
deployment.

The bridge is repo-only and excluded from both Python wheel and sdist. It must
not modify or be represented as part of the Python server, public server
schemas, shared `settings.json`, COMSOL Settings GUI, solver ownership, or
COMSOL compatibility matrix.

## Required repository synchronization

For bridge behavior or compatibility changes, inspect and update together:

- `dsh_bridge/package.json`, implementation, tests, and smoke script;
- `dsh_bridge/README.md` and `dsh_bridge/DEEPSEEK_COMPATIBILITY.md`;
- root `README.md`, `README_CN.md`, and `DEPLOYMENT.md` when user-visible;
- `development_kit/docs/layout.md` for added, removed, or renamed files;
- `pyproject.toml` and release-engineering assertions if package exclusion
  changes.

Keep the adapted upstream acknowledgement for commit `99172f8f43c6753c2442c406cd5c6055ea8c5bef`
in public release documentation.

## Connection and ownership contract

The bridge owns exactly one MCP stdio connection and serializes tool calls,
status polling, tail polling, and cancellation through one single-flight queue.
Do not configure a second `dsh-mcp-client` COMSOL entry in the same DSH session.

The Python server remains authoritative for durable state, journal rows, solver
lease, process identity, source immutability, cleanup, resume, and scientific
evidence. A DSH completion notification is not an independent solve receipt.

`config.enabled` is a DSH Cordis plugin Boolean. Never add it to the COMSOL
Settings GUI or server settings schema. Server profile/settings changes still
require the owning DSH/MCP host lifecycle to restart and capabilities to be
read back. The current user's settings-change route has been independently
accepted, but this does not establish a universal production deployment claim.

## Verification

Run from the repository root:

```powershell
npm test --prefix dsh_bridge
npm run smoke --prefix dsh_bridge
```

The accepted solver-free bridge baseline is 37 Node tests plus the fake-server
smoke. Also run repository release-engineering/layout/package-boundary tests and
the exact-SHA hosted workflow after an authorized push.

For real DSH checks, verify live discovery and `capabilities`, then ownership
status before any solver call. Compare mirrored progress and completion against
the server's durable job state. Report separately:

- fake-server protocol and mirror behavior;
- installed-server transport/discovery behavior;
- settings-change and host-restart behavior;
- production cancellation behavior;
- licensed COMSOL solve behavior.

Never promote one category into another. In particular, 37/37 fake tests do not
prove production cancellation or a licensed solve.

## Failure and recovery

Missing executable, unsupported protocol, malformed discovery, or exhausted
reconnect budget must fail closed. Loss of the bridge process or stdio channel
does not prove that a durable worker stopped. Reconnect, inspect `job_status`,
and use the server's exact `job_resume` contract.

Cancellation is terminal only after the server reports terminal state and the
required cleanup and lease evidence is available. Never replace or restart the
installed server while an active job, lease, COMSOL process, Java owner, or
cleanup uncertainty remains. Preserve bridge state and server receipts; never
rewrite evidence to match a client-side notification.

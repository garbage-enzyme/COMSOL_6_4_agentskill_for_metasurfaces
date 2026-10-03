# DeepSeek Harness bridge development

## Current version boundary

Read the live Python package, installed executable and DSH launcher identities.
Do not infer current versions from historical validation notes. The optional
repository component `dsh_bridge/` is `@local/dsh-comsol-bridge` version `0.1.0`,
requires Node.js 20 or newer, and has no npm dependencies. Its real-host recovery
checks use DSH `0.1.6-alpha.2` with server baseline `0.7.3`. A source change or
bridge push is not a Python production deployment.

The bridge is repo-only and excluded from both Python wheel and sdist. It must
not modify or be represented as part of the Python server, public server
schemas, shared `settings.json`, COMSOL Settings GUI, solver ownership, or
COMSOL compatibility matrix.

## Required repository synchronization

For bridge behavior or compatibility changes, inspect and update together:

- `dsh_bridge/package.json`, implementation, tests, and smoke script.
- `dsh_bridge/README.md` and `dsh_bridge/DEEPSEEK_COMPATIBILITY.md`.
- root `README.md`, `README_CN.md`, and `DEPLOYMENT.md` when user-visible.
- `development_kit/docs/layout.md` for added, removed, or renamed files.
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

Report the current test count from the actual run. The historical 37-test
baseline does not cover later owner-recovery changes. Also run repository
release-engineering/layout/package-boundary tests and the exact-SHA hosted
workflow after an authorized push.

For real DSH checks, verify live discovery and `capabilities`, then ownership
status before any solver call. Compare mirrored progress and completion against
the server's durable job state. Report separately:

- fake-server protocol and mirror behavior.
- installed-server transport/discovery behavior.
- settings-change and host-restart behavior.
- production cancellation behavior.
- licensed COMSOL solve behavior.

Never promote one category into another. Fake-server tests do not prove
production cancellation or a licensed solve.

For restart acceptance, use real Agents and the native job-controller plugin,
an independently persisted fake job, and separate host processes. Verify the
original job/attempt/owner, delayed and missing controllers, session isolation,
progress and cancellation access, terminal mapping, reconnect, persistence
failures, retained blocked records, and no resubmission. Never substitute a
fake registry or a hand-attached controller for this real-host gate.

Native completion delivery and session persistence are separate boundaries.
On DSH `0.1.6-alpha.2`, an unflushed native background-job notice can disappear
after abrupt termination, while public `sessions.flush` preserves it. State
whether acceptance uses native persistence or requires a separately agreed
stronger contract. Do not claim crash-proof or exactly-once delivery from an
in-memory notice. Avoid model turns in solver-free notification tests by using
the native controller's quiet delivery mode.

## Failure and recovery

Missing executable, unsupported protocol, malformed discovery, or exhausted
reconnect budget must fail closed. Loss of the bridge process or stdio channel
does not prove that a durable worker stopped. Reconnect, inspect `job_status`,
and use the server's exact `job_resume` contract.

Terminal jobs found at restart still need their original owner/controller to
receive completion. Retain their records while ownership is unavailable.
Server-terminal state alone is not proof of notification. A known attempt must
match current server status before recovery. Do not attach old tracking records
to a replacement execution or silently rewrite the old identity.

DSH owner/service teardown releases the observer, not independently durable
COMSOL work. Verify public job-hook cancellation reasons against the exact
host version, retain unconfirmed tracking on teardown/controller loss, and keep
explicit user cancellation distinct from lifecycle cleanup. Runtime replacement
must preserve settings/credentials and verify loaded dependencies, not merely
the CLI version. Profile fallback links can retain an older runtime.

Cancellation is terminal only after the server reports terminal state and the
required cleanup and lease evidence is available. Never replace or restart the
installed server while an active job, lease, COMSOL process, Java owner, or
cleanup uncertainty remains. Preserve bridge state and server receipts. Never
rewrite evidence to match a client-side notification.

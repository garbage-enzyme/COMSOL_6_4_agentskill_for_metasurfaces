# Repository and public-contract development

## Contents

- Repository boundaries
- Public tools and profiles
- Typed schemas and boundedness
- Settings contract
- Failure atomicity and cleanup
- Evidence separation
- Windows support boundary

## Repository boundaries

Keep packaged runtime code under `comsol_mcp/`. Treat repository compatibility
layers, tests, scripts, release receipts, fixtures, and recipes as development
content unless packaging metadata explicitly includes them. Runtime code must
not import the development kit or recipes.

Read and update the layout inventory whenever a tracked file is added, removed,
or renamed. Check both wheel and sdist contents; source-tree cleanliness does
not prove artifact cleanliness.

Recipes are self-contained examples, not runtime dependencies. Do not commit
licensed models, private data, credentials, or hard-coded user paths.

## Public tools and profiles

Keep the default profile small and stable. Group tools by capability maturity
and side effect. Discovery must truthfully report whether a tool can construct
a COMSOL client, acquire a lease, start a worker, mutate a model, or load a
large optional dependency.

Use typed, domain-specific operations. Do not expose a generic property setter
as a verified public tool. A public mutation should define exact feature types,
allowed properties, selection form, expression limits, rollback behavior, and
readback evidence.

When a public tool changes, update together:

- catalog registration and static profile membership;
- public input/output schema and limits;
- compatibility snapshots and schema hashes;
- user and developer documentation;
- positive, negative, rollback, and framework-dispatch tests.

Do not use private framework registries or direct wrapped-function calls as the
only proof that public MCP dispatch works.

## Typed schemas and boundedness

Reject unknown fields, non-finite numbers, booleans where numeric scalars are
required, overlong identifiers or expressions, deep/cyclic structures, and
incompatible combinations. Normalize semantically equivalent inputs before
fingerprinting.

Apply bounds before expensive work or side effects. Limit response size,
history, queues, samples, artifacts, journals, descriptors, subprocess output,
and operation-name cardinality. Validate the complete nested object rather than
only top-level fields.

Never return raw exception text from COMSOL or the host as an unrestricted
public response. Map it to stable reason codes and bounded messages while
retaining full private evidence where appropriate.

## Settings contract

Use one project-root `settings.json` shared by all clients. Group settings by
function and keep field meaning in documentation. Missing keys use safe
defaults. An invalid leaf falls back only that leaf and yields a structured,
bounded error; malformed JSON falls back the entire default document.

Do not duplicate profile, runtime, path, Java, shared-server, or evidence
settings across agent-specific configuration files. Pass only the settings-file
locator when the host cannot preserve the project path.

Settings changes require a fresh MCP host. Capabilities and status should expose
effective state, a settings fingerprint, and redacted errors without leaking
private paths.

## Failure atomicity and cleanup

Validate before COMSOL mutation. For a batch create or update, record every
created node and roll back the complete batch on failure. Isolate cleanup steps
so one cleanup error does not suppress later cleanup. Report whether rollback
and cleanup were independently proven.

Treat caller-owned sources as immutable. Mutate only a derived model with a
distinct identity and output. A before/after hash proves equality, not that the
source was never opened for writing; tests should also enforce access mode or
mutation boundaries.

## Evidence separation

Separate execution success, evidence integrity, and scientific disposition.
Changing caller policy must not mutate stored raw evidence. A hash proves
consistency and change detection, not equations, mesh, or physical correctness.

Default-on evidence checks should report explicit opt-outs and propagate an
unverified outcome. Restore a disabled check with a fresh verification; never
upgrade an old receipt in place.

## Windows support boundary

Keep the declared Windows-only support surface explicit in code, tests, CI, and
docs. Use Windows path, process, Job Object, hidden-window, sharing, commit, and
handle semantics intentionally. Do not add superficial POSIX branches that are
untested or weaken Windows guarantees.

Use ASCII runtime, cache, build, and artifact roots when native dependencies or
editable installs are sensitive to non-ASCII user paths. Keep public examples
portable and free of host-specific drive letters or thresholds.

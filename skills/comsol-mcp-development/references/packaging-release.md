# Packaging, release, and production deployment

## Contents

- Namespace and package boundary
- Build and artifact inspection
- Clean non-editable installation
- Build identity and stdio probe
- Exact-SHA release sequence
- Production restart and rollback

## Namespace and package boundary

Use `comsol_mcp` as the packaged public namespace. A repository-only `src`
compatibility layer may support historical imports and source-tree execution,
but it must not enter the wheel or become the production command.

Expose the packaged console entry point as `comsol-mcp =
comsol_mcp.server:main`. Production configuration should use the console script
or `python -m comsol_mcp.server`, never `python -m src.server`.

Exclude tests, development scripts, release plans, private agent configuration,
recipes, caches, bytecode, and licensed assets from the wheel. Inspect the sdist
separately because global ignore rules do not control build-backend inclusion.

## Build and artifact inspection

Build from the exact reviewed source identity. Check whitespace, formatting,
compile, targeted/broad tests, dependency locks, package metadata, and artifact
member lists. Reject unexpected root files and untracked configuration before
release.

Record wheel and sdist SHA-256 values. Validate that every packaged runtime file
is expected and that repository compatibility layers remain absent.

Treat a version bump as one atomic public-identity change. Update package and
GUI versions, bilingual docs, deterministic locale outputs, schema-registry
hashes, release facts, compatibility assertions, and visible header tests
together. Run the repository generators and their `--check` modes; do not hand
edit generated PO/MO or release-facts payloads independently.

## Clean non-editable installation

Install the wheel into a clean environment outside the source checkout. Prefer
ASCII build, cache, runtime, and artifact roots on Windows, especially when the
user profile path contains non-ASCII characters.

Use non-editable installation for release acceptance. If rebuilding the same
version with different content, force reinstall and compare installed identities;
the version string alone cannot prove which code is running. Run `pip check`
and verify the resolved dependency lane.

On Windows, a running console-script process can lock its generated `.exe`.
Before replacing an installed wheel, prove that no COMSOL client or solver
owner is active, identify the target-environment launcher and Python child by
PID, parent PID, command line, and executable path, and stop only that exact
stdio host under caller-authorized deployment or restart. Never terminate by
process name or begin replacement while the launcher lock remains.

If an interrupted uninstall leaves the import missing or pip reports a
`~<distribution>` rollback directory, treat the environment as an incomplete
transaction. Verify each residual path is bounded inside the exact target
`site-packages`, complete the exact-wheel force reinstall with `--no-deps`,
then remove or reversibly quarantine only the verified stale backup paths.
Recheck import location, distribution metadata, generated entries, and `pip
check`; `pip check` alone does not prove that the package import survived.

## Build identity and stdio probe

Discovery should expose redacted identities that distinguish source from
installed package, including package version, tool catalog, public schemas,
profile membership, build manifest, and package-content identity.

Probe installed stdio from a working directory outside the repository. Perform
real MCP initialize, tool discovery, and capabilities calls while proving no
COMSOL client or JVM started. A source-tree import probe is not installed-wheel
acceptance.

For a protocol-compatibility release, probe the exact installed wheel through
raw supported protocol revisions, the oldest declared real client SDK, and the
current configured agent client. Bind every receipt to the same wheel hash and
source SHA, require clean close and zero solver residue, and preserve the
caller's configured model/API provider instead of bypassing user configuration.

## Exact-SHA release sequence

Use this order unless the repository contract is stricter:

1. pass focused and broader local solver-free gates;
2. commit a clean, buildable change;
3. push only with caller authorization;
4. require every CI job for the exact pushed SHA to finish successfully;
5. run the clean locked release gate without a heavy COMSOL workload active;
6. build the final wheel from that exact identity;
7. record hashes and installed-probe receipts;
8. install to production only when the caller authorizes deployment.

Do not infer release readiness from a different commit, a cancelled run, a
rerun that used changed source, or a source-only local pass.

## Production restart and rollback

Migrate effective settings intentionally; do not overwrite production with a
template. Restart every owning stdio client after installation because an
already-running Python process retains old modules in memory.

If a host vanished, validate its configured command before restart. A long-lived
pre-migration host can conceal a stale production command until it exits.
Restart only one stdio host, then verify capabilities and ownership status
serially without starting COMSOL.

Keep the prior reviewed wheel and settings receipt for rollback. If installed
identity, stdio discovery, dependency checks, or settings fingerprint differs
from the release receipt, stop deployment and restore the prior exact package;
do not debug by launching a solver.

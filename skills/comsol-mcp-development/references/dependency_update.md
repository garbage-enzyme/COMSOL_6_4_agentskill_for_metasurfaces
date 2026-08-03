# Dependency maintenance

## Contents

- Interpret drift
- Validate candidates
- Preserve compatibility lanes
- Regenerate the production lock
- Correct the drift report
- Accept or defer

## Interpret drift

Bind every review to the exact source SHA, workflow run, artifact identity, and
input hashes. Do not treat `pip list --outdated` as an upgrade instruction.
Classify normalized PEP 503 distribution names as runtime direct, optional,
development, bootstrap tooling, or runtime transitive.

Distinguish the absolute latest version from the newest version allowed by the
project declaration. Keep an excluded major outside the candidate set. Treat
pip, build, and installer packages as bootstrap tools unless the project
declares them. Diagnose a missing optional package against the workflow's
installed extras before calling it incompatible.

Inspect exact transitive requirements before proposing an independent update.
If one installed package pins another exactly, update the pair only when a
compatible parent release exists. Never force a resolver-invalid pair.

## Validate candidates

Use a fresh ASCII-path environment with the supported standard CPython ABI.
Install the complete review scope, including the development and relevant
optional extras, then run `pip check`. Record the resolver-selected versions,
wheel tags, and hashes. Exercise a newer pip only as lock/build tooling unless
it is a declared project dependency.

Start with the smallest affected solver-free tests:

- MCP imports, server construction, stdio initialization, dispatch, schemas,
  cancellation, and installed entry points;
- rendering arrays, PNG semantics, isolated worker cleanup, and GUI packaging
  for Matplotlib changes;
- strict typing groups and quality inventory for mypy changes; and
- imports, security, license, and SBOM evidence for transitive runtime changes.

Classify a failure before editing. Permit only a narrow project-owned API,
annotation, import, metadata, or rendering adaptation whose public behavior is
unchanged. Defer a dependency that requires an architecture, public schema,
settings, solver, or scientific-behavior change.

## Preserve compatibility lanes

Resolve and test both lanes independently:

1. current-compatible dependencies selected within package metadata; and
2. the unchanged reviewed minimum direct constraints.

Do not raise a minimum merely because the current lane moved. A compatibility
adaptation is acceptable only when both lanes still install, pass `pip check`,
and pass the repository-declared dependency/process suite.

## Regenerate the production lock

Build a wheel from the reviewed source, then generate the complete default
runtime lock with the supported Windows CPython interpreter. Resolve and
download binary wheels once, install those exact wheelhouse bytes, freeze the
environment, and write one hash-pinned entry per runtime distribution. Exclude
the project wheel and bootstrap tools from the runtime lock.

Compare old and new locks package by package. Every changed version and hash
must have a reviewed transitive reason. Verify the normalized lock hash and
requirement count in the tested-versions manifest. Install the lock with
`--require-hashes`, install the project wheel with `--no-deps`, and run
`pip check`, license, vulnerability, SBOM, package, and installed probes.

## Correct the drift report

Keep report construction in deterministic tested code rather than an inline
workflow shell block. Preserve source SHA, Python/ABI/platform, input paths and
hashes, installed inventory, raw outdated inventory, and `pip check` outcome.
Report exact project scope, declared specifier, reviewed/locked/installed
versions, absolute latest, allowed latest and its basis, paired requirements,
complete lock drift, and a stable decision such as `candidate`,
`outside_project_range`, `paired_only`, `informational`, or `missing_scope`.

The workflow remains information-only: no automatic dependency edit, lock
rewrite, branch, pull request, merge, or release.

## Accept or defer

After focused success, run the repository's complete local solver-free split,
quality gate, GUI/package gates when affected, and clean installed stdio probes.
Commit and push only with caller authorization, then require every job for the
exact pushed SHA. A local pass does not accept the lock, and a different CI SHA
does not accept the candidate.

Stop when the resolver crosses an excluded major, violates an exact pair,
changes an unapproved minimum, lacks a supported wheel, changes public or
scientific behavior, produces an incomplete/unhashed lock, or starts
COMSOL/Java during solver-free maintenance.

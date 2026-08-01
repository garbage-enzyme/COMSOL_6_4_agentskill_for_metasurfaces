# Testing, CI, and gate development

## Contents

- Test layers
- Solver-free enforcement
- Local and hosted execution policy
- Deterministic time and warnings
- Failure diagnosis
- Quality gate and release gate
- Long-run waiting and stability

## Test layers

Use the smallest layer that proves the behavior, then expand by risk:

1. deterministic unit tests with fake clocks, fake processes, and injected I/O;
2. Windows process-only tests for sharing, identity, Job Objects, pipes, and
   cleanup;
3. repository integration tests for stdio, package boundaries, schemas, and
   receipts without COMSOL;
4. opt-in licensed COMSOL acceptance for release-specific clientapi or physical
   behavior.

Tests should assert observable behavior and independent evidence, not private
implementation metadata. Include accepted boundary equality, one-step-beyond
rejection, rollback, cleanup, replay, and corruption cases.

## Solver-free enforcement

Unit, lint, compile, documentation, schema, wheel, sdist, and default CI must not
construct `mph.Client`, start Java, or launch COMSOL. Use explicit observers or
receipts to prove `solver_started=false` rather than relying on convention.

Keep native-backed imports that can stall the event loop either preloaded before
dispatch or isolated in bounded subprocesses. A fresh-stdio test matrix should
cover representative cold paths.

## Local and hosted execution policy

Follow the current repository `AGENTS.md` commands exactly. Keep the measured
local worker count bounded and retain any declared serial tail for startup or
process-inventory tests. Do not replace it with `-n auto` without a new timing,
isolation, cleanup, and coverage benchmark.

GitHub-hosted Windows execution may require serial pytest even when local xdist
is stable. Different hosted jobs have stalled during worker dispatch or
shutdown. Do not restore hosted xdist until an upstream fix or a new repeated
stability benchmark justifies it.

Never start a full local suite or gate while a heavy COMSOL solve is active. A
heavy solve may exhaust the host alone; overlap only increases the risk.

## Deterministic time and warnings

Keep ordinary unit tests independent of the wall clock. Pin the review date or
derive it from a checked-in fixture. Enforce “current as of today” in a dedicated
CI or release command gate whose purpose is freshness.

Treat `ResourceWarning` as lifecycle evidence. Close files, SQLite connections,
pipes, subprocess streams, executors, and temporary resources deterministically.
Run focused suites with warnings promoted to errors when repairing cleanup.

For timing tests, use fake clocks when possible and independent absolute
ceilings when real scheduling is unavoidable. Do not tighten a relative timing
assertion until it survives process-enumeration and hosted-runner load.

## Failure diagnosis

Bind diagnosis to the exact commit SHA, workflow run, job, and step. Extract the
first actual traceback or failed assertion instead of inferring from elapsed
time or a progress percentage.

Classify separately:

- deterministic assertion/schema/allowlist mismatch;
- timeout with diagnostic stack;
- worker or executor shutdown stall;
- process/pipe/resource leak;
- host resource exhaustion;
- dependency-lane incompatibility;
- packaging or installed-source mismatch.

One deterministic failure repeated in several jobs is one root cause, not
several independent CI defects. Update generated fingerprints or allowlists only
after proving the matched semantic surface is intentionally unchanged.

## Quality gate and release gate

The quality gate runs formatting/lint, the solver-free test split, coverage,
license, budget, and receipt checks in the working tree. It is a development
gate and may permit an explicitly identified dirty tree.

The release gate proves a clean, locked, reproducible release candidate. It
should verify dependency locks, wheel/sdist contents, clean non-editable install,
installed schemas/profiles/build identity, stdio discovery, and absence of a
COMSOL client. A quality pass does not substitute for the release gate.

## Long-run waiting and stability

For a test or gate expected to exceed three minutes, wait once for the current
ETA plus a small margin instead of frequent polling. If it remains active,
derive a new ETA from observed progress and repeat the bounded wait.

Do not call an intermittent stall fixed after one pass. Establish a fresh
post-fix baseline and require the repository-declared consecutive exact-SHA CI
window. Distinguish a fast deterministic failure from a no-progress stall when
counting stability.

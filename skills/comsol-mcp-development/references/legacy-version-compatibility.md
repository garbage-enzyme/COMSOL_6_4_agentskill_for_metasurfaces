# Explicit legacy COMSOL compatibility

## Contents

- Trigger and scope
- Prove the compatibility conflict
- Establish a reversible baseline
- Implement the narrowest adapter
- Validate without expanding claims
- Stop conditions
- Roll back to the public GitHub version

## Trigger and scope

Use this workflow only when all conditions hold:

1. the exact detected COMSOL release is older than 6.4; and
2. the user explicitly asks the agent to attempt compatibility with that
   release; and
3. the evidence in the next section proves a reproducible conflict caused by a
   specific difference between that release and the supported reference build.

Until all three conditions hold, diagnose and collect bounded evidence only.
Do not edit MCP code, dependencies, schemas, settings, or support claims on the
basis of version age, a generic native exception, or an assumed API difference.

Do not infer authorization from an old model file, an installed executable, or
a generic request to “make it work.” Do not change the advertised support range,
dependency bounds, profiles, or public schemas merely because one legacy build
is present.

Bind the attempt to one exact COMSOL build, operating system, MCP commit, tool or
workflow, and reproducible conflict. Success for one build or feature is not a
claim that all earlier COMSOL versions are supported.

## Prove the compatibility conflict

Require concrete evidence before editing:

- exact COMSOL display and executable versions read from authoritative runtime
  surfaces;
- exact public MCP source/build identity and settings fingerprint;
- the smallest request that fails on the legacy release;
- complete bounded error type, stable message/reason code, failing API call, and
  lifecycle phase;
- expected behavior from the current public contract;
- reflection, feature/property inventory, official release documentation, or a
  controlled probe showing the legacy API difference.

The evidence must connect the failing operation to one exact incompatibility,
such as a missing or renamed typed API, a changed overload, a different result
shape, or a documented behavior change. A failure that merely occurs on an old
release is correlation, not compatibility evidence. If the causal difference
cannot be identified, stop without a code change and classify the result as
unproven.

Distinguish an actual version conflict from licensing, missing modules, wrong
profile, invalid model topology, stale deployment, solver collision, resource
exhaustion, or an unrelated code defect. If the same failure occurs on the
supported reference build, repair it as a general defect instead of a legacy
compatibility patch.

## Establish a reversible baseline

Start from the exact publicly available GitHub tag or commit currently deployed.
Record the repository URL, commit SHA, release artifact hash, installed package
identity, settings receipt, and stdio discovery receipt.

Create a separate compatibility branch or clean worktree. Preserve unrelated
user changes and source models. Keep the last public wheel and effective settings
available for rollback. Do not overwrite the only copy of a working production
installation or use a destructive reset to prepare the experiment.

Run solver-free baseline tests and an installed-package stdio probe before the
legacy attempt. Use a distinct ASCII artifact/runtime root for compatibility
evidence.

## Implement the narrowest adapter

Before editing, state the proven conflict, the exact code and contract surfaces
that may change, the exact requested build, and explicit non-goals. Patch only
that declared change envelope. Prefer a small version-gated adapter around an
exact overload, feature type, property name, selection rule, or result shape.
Keep the 6.4 path unchanged and make the legacy branch explicit and testable.

Do not:

- add broad exception swallowing or retry every native error;
- probe many undocumented properties by mutation;
- weaken validation, rollback, ownership, evidence, or cleanup requirements;
- expose arbitrary property setters as a compatibility escape hatch;
- silently substitute another physics interface or numerical meaning;
- combine the adapter with unrelated refactoring, dependency upgrades, feature
  work, cleanup, or speculative hardening;
- expand compatibility to other versions without their own evidence.

If the legacy API cannot represent the public contract faithfully, return an
explicit unsupported result rather than approximating success.

## Validate without expanding claims

Require all of the following before accepting the narrow patch:

- deterministic fake/reflection tests for the exact legacy difference;
- regression coverage proving the supported 6.4 path is unchanged;
- clean package, schema, profile, and solver-free stdio gates;
- an opt-in licensed acceptance on the exact requested legacy build when native
  behavior is part of the claim;
- source-model immutability, rollback, cleanup, and ownership evidence;
- a receipt whose compatibility statement names only the tested build, tool,
  workflow, and limitations.

Do not update general README support claims from a smoke test. Publish a scoped
compatibility note only after the exact legacy gate and ordinary release gates
pass.

## Stop conditions

Stop the compatibility attempt when any of these occurs:

- the required native API or physics feature does not exist and no typed,
  semantically equivalent route is available;
- the patch would weaken safety, evidence integrity, source immutability,
  ownership, rollback, or boundedness;
- the legacy result cannot be distinguished from a false-success approximation;
- the exact licensed legacy gate cannot be run or repeatedly fails after a
  bounded, evidence-driven repair sequence;
- cleanup, process ownership, model state, or installed identity becomes
  uncertain;
- the requested change would require broad support claims beyond the proven
  conflict.

Classify the attempt as unsupported or blocked with the exact evidence. Do not
continue by changing unrelated parameters or widening exception handling.

## Roll back to the public GitHub version

On an unfixable or unsafe failure, restore production to the exact public GitHub
version recorded at baseline:

1. stop only compatibility-owned MCP/test/solver processes and preserve failure
   artifacts;
2. leave user-owned COMSOL, Desktop, Server, launchers, models, and unrelated
   worktree changes untouched;
3. reinstall the recorded public wheel non-editably, or rebuild it from the
   recorded public tag/commit and verify its hash;
4. restore the recorded effective settings and packaged console entry point;
5. restart one fresh stdio host without starting COMSOL;
6. run dependency checks, installed build/schema/profile identity checks, MCP
   initialize/tool discovery, and capabilities/status serially;
7. write a rollback receipt binding the failed attempt, restored public commit,
   artifact hash, settings fingerprint, and solver-free verification.

Preserve the compatibility branch for analysis unless the user asks to delete
it. Rollback means restoring the deployed public version, not erasing evidence or
resetting the user's repository.

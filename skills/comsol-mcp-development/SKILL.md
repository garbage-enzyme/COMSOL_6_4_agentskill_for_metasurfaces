---
name: comsol-mcp-development
description: Develop, review, test, package, release, deploy, and recover the Windows-only COMSOL Multiphysics MCP server. Use when editing the COMSOL MCP repository or its tools, schemas, profiles, settings, durable jobs, evidence contracts, tests, CI workflows, recipes, packaging, wheel installation, production stdio host, or when diagnosing Windows races, process leaks, resource exhaustion, CI stalls, installed-source mismatches, and release-gate failures. Do not use for ordinary COMSOL modeling or scientific result validation; use the COMSOL operations skill for those tasks.
---

# COMSOL MCP development

Use this skill for server engineering. Keep ordinary modeling, solving, and
physical-validation guidance in the separate COMSOL operations skill.

## Mandatory workflow

1. Read the repository `AGENTS.md`, then the layout document and the closest
   implementation, test, schema, and contract before editing.
2. Treat the live repository and installed-package identities as authority.
   Historical plans explain why a rule exists but do not override current code.
3. Keep unit, schema, packaging, documentation, and process-only work
   solver-free. Real COMSOL checks are explicit, licensed, serial gates.
4. Preserve unrelated worktree changes. Stage only the intended files and do
   not commit or push without caller authorization.
5. Keep internal review identifiers and private plan labels out of source,
   commit messages, branches, CI output, release notes, and public docs.
6. Update public schemas, profiles, settings docs, snapshots, layout inventory,
   and release facts together whenever their observable contract changes.
7. Distinguish deterministic code failures, timing races, process leaks,
   resource exhaustion, transport loss, and stale deployment before selecting a
   fix. One error can expose another latent defect.

## Reference router

Read each selected reference completely before acting.

| Task | Read |
| --- | --- |
| Repository layout, public tools/profiles/schemas, settings, bounded inputs, failure atomicity, evidence separation | [repository-contracts.md](references/repository-contracts.md) |
| Durable jobs, atomic state, ownership, process identity, cancellation, admission races, Windows sharing, resource exhaustion, MCP host recovery | [runtime-reliability.md](references/runtime-reliability.md) |
| Unit/stress/integration layers, local and hosted pytest policy, warnings, deterministic clocks, CI diagnosis, quality and release gates | [testing-ci.md](references/testing-ci.md) |
| Package boundaries, `comsol_mcp` namespace, wheel/sdist, non-editable install, build identity, exact-SHA release, production restart | [packaging-release.md](references/packaging-release.md) |
| Fresh-install disk sizing, package/dependency attribution, optional-extra boundaries, and temporary-space estimates | [deployment-sizing.md](references/deployment-sizing.md) |
| Dependency drift, direct/optional/dev/bootstrap classification, paired constraints, dual-lane validation, lock regeneration, and report workflows | [dependency_update.md](references/dependency_update.md) |
| Explicitly requested COMSOL versions below 6.4, conflict proof, narrow compatibility patches, stop rules, and rollback to a public GitHub version | [legacy-version-compatibility.md](references/legacy-version-compatibility.md) |
| External-review reconciliation, parent/child verification, dispositions, repair ordering, commit boundaries, TODO and receipt maintenance | [review-maintenance.md](references/review-maintenance.md) |
| Optional DeepSeek Harness bridge, DSH installation, single-connection ownership, job mirroring, settings-change verification, and bridge evidence boundaries | [deepseek-harness-bridge.md](references/deepseek-harness-bridge.md) |

## Default change sequence

1. Reproduce or prove the defect with the smallest solver-free test.
2. Identify the public contract and failure boundary before changing code.
3. Implement the narrowest fix and add deterministic regression evidence.
4. Run focused tests, then the repository-declared broader gate in the declared
   environment. Do not invent a new parallelism or timeout policy mid-fix.
5. Inspect the complete diff, formatting, generated/snapshot files, and package
   boundary. Update the development ledger or TODO before handoff.
6. Commit a buildable cohesive batch. After an authorized push, bind the result
   to the exact pushed SHA and inspect every required CI job.
7. Build and deploy only from the exact accepted source identity. Verify the
   installed package outside the checkout before restarting the stdio host.

## Outcome language

Use `reproduced`, `deterministic_failure`, `timing_race`, `resource_exhaustion`,
`transport_failure`, `deployment_mismatch`, `implemented_pending_ci`, `fixed`,
`rejected`, and `deferred` precisely. A local pass is not exact-SHA CI success;
a source-tree pass is not production deployment; a restarted MCP host is not a
successful COMSOL solve.

## Reusable maintenance patterns

- For Windows Settings GUI modal progress views, reserve the action footer as
  an independent bottom-packed region, bound the dialog to the monitor, and
  truncate untrusted dynamic source labels for display. High-DPI and long-name
  Tk scenarios must verify that cancellation remains reachable.
- When an optional public feature is renamed or split, update its catalog gate,
  settings migration, environment mapping, profile/feature snapshots, release
  facts, quality-target inventory, GUI visual golden, and legacy compatibility
  assertions in the same change. Run the repository quality gate after focused
  tests because stale counts can survive narrow GUI coverage.
- For wavelength-controlled one-point evidence, retain and compare all three
  identities: caller-requested value, evaluated model parameter, and solved
  frequency-derived wavelength. Equality between only the latter two can hide
  a study sweep overriding the requested point; use the study's explicit
  single-point parameter control and fail closed on any requested-value drift.
- For subprocess protocols carried on stdout, treat every imported dependency
  as a possible protocol writer. Prefer the dependency's current module entry
  point over deprecated compatibility imports, keep diagnostics on stderr, and
  test the child in a fresh optional-dependency environment while parsing every
  stdout line as the declared protocol.
- For PDF/manual indexing workers, reserve stdout exclusively for framed JSON
  events. Duplicate the protocol fd before redirecting native-library stdout
  to bounded stderr; otherwise native PDF diagnostics can masquerade as invalid
  JSON and cause a false worker failure. Verify the real corpus, preserve the
  previous index on cancellation/failure, and record only bounded relative
  source/page diagnostics.
- For native COMSOL adjoint adapters, prove the objective can be assembled by
  the native sensitivity solver. A forward-valid aggregate such as `Ttotal`
  may be undefined in the adjoint objective scope; use a documented
  differentiation-aware order expression when required and bind the generated
  sensitivity solution/dataset identities. Keep raw complex `fsens` evidence
  separate from the accepted real derivative component, which requires an
  independent finite-difference check.
- For licensed native optimizer gates, bind the resource iteration cap, exact
  solver iteration request, method, and move limit as separate caller-owned
  inputs and verify solver readback. If a later shape step produces deformation
  or material-coordinate NaNs, preserve the accepted trajectory and failed
  evidence, replay bounded next-step fractions with fresh forward solves, and
  audit frame/Jacobian feasibility before changing policy. A smaller move-limit
  causal pass is model evidence, not a universal default; never hide an
  automatic reduction, iteration truncation, or method fallback.
- The `comsolless_read_only` profile is frozen at five stdlib-only read-only
  tools (mph_inspect, mph_diff, model_identity,
  runtime_compatibility_status, offline_export_validate). Any change must
  keep members solver-free with no-heavy-import cold-discovery proof in the
  public-surface suite, update catalog/schema snapshots/release facts in the
  same batch, re-derive the frozen lint-exclusion digest when targeting new
  production modules, and respect the hash/synchronization gate for the
  user-facing operations reference that lives only in the operations skill's
  references folder — never duplicate that usage guide here.

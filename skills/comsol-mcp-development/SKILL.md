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
| Dependency drift, direct/optional/dev/bootstrap classification, paired constraints, dual-lane validation, lock regeneration, and report workflows | [dependency_update.md](references/dependency_update.md) |
| Explicitly requested COMSOL versions below 6.4, conflict proof, narrow compatibility patches, stop rules, and rollback to a public GitHub version | [legacy-version-compatibility.md](references/legacy-version-compatibility.md) |
| External-review reconciliation, parent/child verification, dispositions, repair ordering, commit boundaries, TODO and receipt maintenance | [review-maintenance.md](references/review-maintenance.md) |

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

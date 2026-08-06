# Review reconciliation and maintenance workflow

## Contents

- Freeze source and report identity
- Reconstruct review hierarchy
- Verify claims
- Order repairs
- Maintain the ledger
- Commit and CI boundaries
- Public communication

## Freeze source and report identity

Bind the review report, project summary, source commit, branch, and clean/dirty
state before validation. A review generated from another source identity is
context, not proof about the current tree.

Inspect the report format and parser documentation instead of guessing how
summary parents map to detailed findings. Preserve malformed, duplicated, or
unmapped entries as explicit reconciliation results.

## Reconstruct review hierarchy

Verify summary-level parents first, then locate and inspect each mapped detailed
claim. Treat quick wins, hotspots, cross-cutting concerns, and other summary
families the same way. After every parent-associated child is reconciled, audit
the remaining standalone findings separately.

Do not assume a parent is valid because several children are valid, or reject a
child because its parent is overstated. Record canonical ownership when one
detail supports several parents so it is fixed and counted once.

## Verify claims

For each claim, inspect the exact code path, caller, contract, tests, and runtime
semantics. Prefer deterministic solver-free proof. Use real COMSOL only when the
remaining assertion is release-specific or physical and the caller explicitly
permits the licensed gate.

Use dispositions such as:

- `confirmed` — defect exists and is actionable;
- `rejected` — claim conflicts with code or declared scope;
- `duplicate` — another canonical claim owns the same defect;
- `deferred` — valid but outside the current release boundary;
- `implemented_pending_ci` — fix and local gates pass;
- `fixed` — required exact-SHA CI and release boundary pass.

Do not mark a claim fixed because code changed or one focused test passed.

A reviewer disposition is evidence to check, not permission to edit. Reproduce
the exact premise against the current source before every repair. If the path is
already guarded, unreachable, outside declared support, or correct by design,
change the disposition to `rejected` and preserve the behavior; never weaken a
correct contract merely to exhaust a review count.

For test-oracle findings, require the reviewer to name a concrete mutation that
would still pass while violating the public contract. Direct equality is valid
for intentional schema, enum, generated-output, and release-identity goldens.
Reject circular expected values only when expected and actual genuinely share
the same implementation path.

## Order repairs

Finish release-blocking summary parents before unrelated standalone findings
unless the caller explicitly overrides the order. Within one queue, rank by
severity, public reachability, data/solver safety, dependency, and repair risk;
the identifier prefix alone does not determine priority.

Group claims with one root cause into a cohesive buildable commit. Respect the
live plan's current batch and push cadence. A CI-only correction may be pushed
immediately when needed to return the exact source window to a valid state.

## Maintain the ledger

Keep one authoritative plan as the progress ledger. At the top, record branch,
base identity, execution rules, stop conditions, and current release boundary.
Maintain explicit TODO checkboxes for implementation, tests, commit, push,
exact-SHA CI, release gate, artifact build, deployment, and final receipt.

For already verified parent-associated details, keep only identifiers when full
descriptions would make the plan unusable. Keep standalone verified items in a
separate final section with disposition and a one-sentence explanation.

Update the ledger after local verification, after commit/push, after CI, and
after deployment. Preserve failed runs and superseded receipts rather than
rewriting history.

Keep review-tool false-positive rules outside the product repository unless
they are intentionally public project policy. Order specific path rules before
general fallbacks when the tool uses first-match semantics, then validate JSON
and representative matches with the tool's own rule-inspection command.

## Commit and CI boundaries

Before commit, inspect the staged diff, run the declared focused/broader gates,
and ensure unrelated changes remain unstaged. Commit messages describe the
public defect and remedy without internal report codes.

Before push, verify network access and the target remote/branch. Bind CI to the
full pushed SHA. A missing, cancelled, failed, or different-SHA workflow is not
success. Record job durations and failure class only when they help future
diagnosis.

## Public communication

Keep usernames, private paths, host thresholds, proprietary models, internal
review IDs, and unpublished project data out of the repository and release
notes. Public notes should describe observable behavior: safer Windows state
publication, deterministic license checks, stronger admission races, stable
cleanup, or corrected package entry points.

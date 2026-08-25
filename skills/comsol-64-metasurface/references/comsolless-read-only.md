# COMSOL-less read-only profile: offline inspection without COMSOL

This reference is the single authority for using the `comsolless_read_only`
profile on a lightweight host that has no COMSOL installation. It assumes
nothing about solvers, licenses, or Java, and every tool it describes is
pure stdlib/offline.

## Host prerequisites

- Supported Python (see the installed package's declared range) and a
  non-editable install of the matching `comsol-mcp` package.
- COMSOL Multiphysics, any Java runtime, an MPh client, JPype, a license,
  solver ownership, and `COMSOL_*`/`JAVA_HOME`-style environment variables
  are **not** prerequisites. None of the five tools imports or starts them.
- No solver lease is acquired and no process is created by this profile.

## Profile selection and restart

- Set the profile to `comsolless_read_only` in the shared `settings.json`
  (`profile.name`) or via the GUI Profile tab, then restart the MCP host.
  Like every profile change, the new surface exists only after restart.
- The base surface is exactly five tools — `mph_inspect`, `mph_diff`,
  `model_identity`, `runtime_compatibility_status`,
  `offline_export_validate` — and nothing else. No session, model, solve,
  job, or knowledge tool can be reached from this profile. Optional
  default-off feature gates compose as everywhere else but cannot enable a
  solver-bound tool because the base set contains none.
- Membership grew one tool per landed package through D1–D6 and is frozen at
  these five from D6 onward.

## Progressive availability of the five tools

| Package | Tool |
| --- | --- |
| D1 | `mph_inspect` |
| D2 | `mph_diff` (+ warning-only post-run artifact probe) |
| D3 | `model_identity` |
| D5 | `runtime_compatibility_status` |
| D6 | `offline_export_validate` |

## Tool semantics you can rely on

- `mph_inspect`: bounded stdlib ZIP reading; reports only what archive
  markers declare (version, node/runnable/solved/preview state, parameters,
  physics/study/material/solution/geometry/mesh tags, savepoints,
  deterministic size breakdown). Ambiguous or conflicting markers fail
  closed with typed reason codes.
- `mph_diff`: binds both inputs, re-hashes after comparison, fails closed on
  mid-diff mutation, and never calls a zero diff a scientific equivalence
  proof.
- `model_identity`: file/source/derived/checkpoint identity plus runtime
  distribution versions from package metadata. Live session identity is
  structured-unavailable unless explicitly requested; requests never attach,
  start, or own anything. Conflicting declared identity yields
  `pause_and_repair`, not permission to continue.
- `runtime_compatibility_status`: active Python/MPh/JPype from package
  metadata, supported ranges from the frozen manifest, profile and skill-layer
  hashes, warnings for detected-but-unsupported installs; bound COMSOL build
  only on explicit request against an already-connected session; never picks
  a fallback runtime or profile.
- `offline_export_validate`: validates an export manifest and its CSV/TXT/VTU
  bytes (containment, IDs, ordering seal, units, counts, hashes) fully
  offline; unsupported reader claims are rejected.

## Safety limits enforced everywhere

Absolute-path and lexical-containment checks, ZIP entry guards (traversal,
absolute/backslash names, duplicates case-folded, symlinks, encryption),
compression-ratio and byte-size caps, entry-count caps, UTF-8 marker
validation with DTD/entity refusal, path redaction in every public field,
bounded row counts, and canonical SHA-256 fingerprints for determinism.

## Three different kinds of evidence

1. **Archive metadata** — what markers declare inside a `.mph`.
2. **Export integrity** — manifest plus artifact bytes/hashes/units/order.
3. **Scientific FEM evidence** — solver receipts, closure, mesh, and physics
   acceptance from durable licensed runs.

Tools here produce kinds 1 and 2 only. Nothing in this profile promotes an
export or an inspection to FEM validation.

## Expected failure codes

- Missing/corrupt/unsupported archives: `mph_source_unavailable`,
  `mph_invalid_zip`, `mph_unsupported_format`, `mph_file_too_large`,
  `mph_too_many_entries`, `mph_duplicate_entry_name`, `mph_entry_path_unsafe`,
  `mph_symlink_like_entry`, `mph_encrypted_entry`,
  `mph_excessive_compression_ratio`, `mph_marker_dtd_rejected`,
  `mph_marker_xml_invalid`, `mph_marker_encoding_invalid`,
  `mph_version_marker_ambiguous`, `mph_conflicting_version_markers`,
  `mph_runnable_state_ambiguous`.
- Identity/checkpoint: `declared_file_hash_mismatch`,
  `source_unavailable`, `checkpoint_missing/unavailable/empty`,
  `declared_checkpoint_hash_mismatch`, `identity_session_provider_invalid`.
- Exports: `manifest_unavailable`, `manifest_not_valid_json`,
  `stale_manifest_hash`, `ordering_drift`, `path_escape`,
  `duplicate_artifact_id`, `missing_file`, `byte_count_mismatch`,
  `artifact_hash_mismatch`, `too_many_artifacts`,
  `model_identity_mismatch/undeclared`, `manifest_invalid`.
- Live-session requests on a host with no session: structured unavailable
  results (`no_connected_session`, `no_active_tracked_model`) — never an
  attach attempt and never guessed values.

## Lightweight install, cold discovery, and example flows

Install non-editably into any supported Python on an ASCII path; no COMSOL
steps exist for this profile. Cold discovery serves exactly the five tools.
Typical flows:

1. Inspect: `mph_inspect {"file_path": "D:/models/probe.mph"}` → read the
   summary fingerprint, version, state, parameters.
2. Compare two saves: `mph_diff {"left_path": ..., "right_path": ...}` →
   metadata/tag/parameter/entry changes with input-immutability proof.
3. Bind identity:
   `model_identity {"file_path": ..., "source_path": ..., "checkpoint_path":
   ..., "expected_checkpoint_sha256": ...}` → checkpoint_ready plus typed
   failure reasons when anything conflicts.
4. Runtime report: `runtime_compatibility_status {}` → active identities,
   ranges, skill hashes, warnings.
5. Validate exports: `offline_export_validate {"manifest_path": ...}` →
   per-artifact verdicts; integrity evidence stays distinct from FEM proof.

Proof that nothing solver-side runs: a fresh interpreter serving this profile
keeps `mph`, `jpype`, and ML packages out of `sys.modules` while all five
tools answer (covered by the repository's public-surface suite).

## Explicit non-goals

Modeling, mutation, solving, job submission, session management, Desktop
attach, Java Shell, HTTP serving, generic command execution, automatic
recovery, and scientific acceptance are all out of scope for this profile.

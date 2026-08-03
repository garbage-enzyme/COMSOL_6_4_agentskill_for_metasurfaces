# Settings GUI

Use this module for installed Settings GUI onboarding, profile selection,
editing ownership, startup diagnostics, and post-save verification. Keep future
GUI workflows here rather than expanding the short skill entry point.

## Open and hand off

When `capabilities.project_settings.setup_required` is true, or the user asks to
edit startup settings, offer the installed GUI or an agent-edited JSON workflow.
For the GUI path:

1. Call the profile-independent solver-free `settings.start` tool once. Treat
   both `launched` and `already_running` as a successful handoff.
2. Tell the user that changes take effect only after restarting Codex or the
   owning MCP client.
3. Stop further task work and wait for the user's next message while the GUI is
   open.
4. Do not edit `settings.json` directly, call `settings.start` repeatedly, or
   start COMSOL/Java while the user owns the GUI transaction.

Use the installed `comsol-mcp-settings` entry point only as the direct fallback
when the agent-facing MCP tool is unavailable.

## Profile guidance

Use `core` as the limited safety-first default for a new user. Recommend
`basic_fem` for most ordinary COMSOL modeling unless the requested workflow
requires another declared profile. Treat the GUI descriptions and live
`capabilities` result as authoritative for every other profile.

## Validation and compatibility

The GUI accepts bounded `_comment*` keys from readable legacy settings as
non-editable metadata; do not ask the user to delete them. Known invalid fields
must be visibly identified and keep Save and Apply disabled. Missing, damaged,
future-version, or structurally unsupported settings require the explicit
rebuild-or-exit workflow rather than a partial rewrite.

If write actions are disabled without a highlighted editable field, inspect the
installed package identity and the settings validation result without changing
the file. Treat a failure caused only by accepted legacy metadata as a stale
deployment or compatibility defect, preserve the settings bytes, and repair the
package before editing settings.

## Finish and verify

After the user applies or saves changes:

1. Ask the user to restart Codex or the owning MCP client; an existing host does
   not hot-load settings.
2. Call `capabilities` on the fresh host before any COMSOL start.
3. Verify the installed deployment identity, configuration state, settings
   fingerprint, selected profile, and bounded settings errors.
4. Resume solver work only after the fresh host reports the intended effective
   configuration.

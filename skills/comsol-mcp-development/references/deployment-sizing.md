# Deployment disk sizing

Measure deployment size from the accepted installed artifact rather than from
dependency names or wheel download sizes.

## Fresh-install method

1. Use the clean release-gate venv that installed the accepted wheel and locked
   default runtime dependencies.
2. Create an empty venv with the same Python executable and environment policy.
3. Sum regular-file logical bytes in both trees.
4. Report the installed-minus-empty delta as the default runtime installation
   footprint and the complete installed venv as the steady-state total.
5. Attribute distributions using installed metadata `RECORD` entries and their
   live file sizes. Keep the MCP package, default dependencies, and base tools
   such as pip separate.
6. Remove the disposable baseline only after resolving and verifying that its
   absolute path is inside the approved test root.

## Report boundaries

Distinguish all of these values:

- compressed wheel size;
- installed MCP package files;
- default runtime dependencies;
- empty Python/venv baseline;
- complete steady-state venv;
- temporary download, extraction, and pip-cache headroom;
- optional extras such as manual or semantic search;
- user data such as PDF manuals, lexical/vector indexes, models, and evidence;
- COMSOL Multiphysics and Java, which are external products rather than Python
  package dependencies.

Do not encode one workstation's measured sizes as product requirements. Package
versions, filesystem cluster size, Python distribution, optional extras, pip
cache policy, and preinstalled shared dependencies change the result. Report
the exact measured environment and recommend conservative free-space headroom;
for a few-hundred-MiB default runtime, one GiB of free space is a practical
installation allowance, not a contractual minimum.

Large scientific stacks usually dominate. Rank actual installed distributions
before proposing dependency removal: SciPy/NumPy and Matplotlib's font/image
stack can outweigh the MCP package by an order of magnitude, but they may be
required public runtime dependencies rather than removable waste.

# COMSOL MCP Skill — 6.3 ClientAPI Operations

English | [中文](README_CN.md)

An agent skill that teaches AI coding assistants how to drive **COMSOL Multiphysics 6.3** through the [comsol MCP server](https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_3_Calibrated) (MPh 1.3.1 standalone / `clientapi`), and how to write or fix code in `src/tools/` when the API mismatches.

## Compatible with: opencode, Claude Code, Codex (and any AGENTS.md reader)

This repo uses the open agent-skill conventions so the **same** skill content works across three AI coding tools without duplication:

| Tool | Entry file it reads | How the skill loads |
| --- | --- | --- |
| **opencode** | `skills/comsol-63-operations/SKILL.md` | Auto-loaded from `~/.config/opencode/skills/` when the task matches the frontmatter `description` |
| **Claude Code** | `CLAUDE.md` (project root) | `CLAUDE.md` contains `@skills/comsol-63-operations/SKILL.md`, an import that injects the full skill body into context |
| **Codex CLI** (and any AGENTS.md reader: Gemini CLI, Cursor) | `AGENTS.md` (project root) | `AGENTS.md` is read automatically; it instructs the agent to open `skills/comsol-63-operations/SKILL.md` for COMSOL tasks |

The single source of truth is `skills/comsol-63-operations/SKILL.md` (markdown + YAML frontmatter). `CLAUDE.md` and `AGENTS.md` are thin pointers — no content is duplicated.

## What the skill covers

- **clientapi vs direct-Model differences** — `tags()` iteration vs int index, `feature().size()` vs `len()`, `getNBoundaries()` capitalization, `physics().create(tag, type, sdim_String)` three-arg signature, study step type full names, `mesh.getNumElem()`, `jm.study().run()` instead of `m.study().run()`.
- **6.3 Electrostatics traps** — default `fsp1` (FreeSpace) domain feature ignores material `relpermittivity`; must add `ChargeConservation` + material node. Block boundary numbering is NOT 1–6 ↔ −x/+x/−y/+y/−z/+z (bnd 3 = z=0, bnd 4 = z=max). `Terminal` `V0` doesn't pin voltage — use `ElectricPotential`. Expression: `1[V]^2` is a parse error, must be `(1[V])^2`.
- **Mesh & study pitfalls** — COMSOL doesn't auto-create a mesh sequence; study step type must use full names (`Stationary`, not `stat`).
- **Capacitance verification recipe** — pure-Python (mph Client) and MCP-tool-flow end-to-end recipes. Verified: **C = 1.8593794414 pF** vs theory 1.8593794407 pF (err 4 × 10⁻¹⁰ pF).
- **Debugging tips** — JPype reflection to probe clientapi overloads, `Box` selection to identify boundary numbers by coordinate, `m.evaluate('V')` field arrays.

## Install

### Option A — opencode

opencode auto-loads skills from `~/.config/opencode/skills/` (`%USERPROFILE%\.config\opencode\skills\` on Windows). Copy the skill folder there:

```bash
git clone https://github.com/garbage-enzyme/COMSOL_6_3_mcp_skill.git
mkdir -p ~/.config/opencode/skills
cp -r COMSOL_6_3_mcp_skill/skills/comsol-63-operations ~/.config/opencode/skills/
```

Windows PowerShell:

```powershell
git clone https://github.com/garbage-enzyme/COMSOL_6_3_mcp_skill.git
$dest = "$env:USERPROFILE\.config\opencode\skills"
New-Item -ItemType Directory -Path $dest -Force | Out-Null
Copy-Item -Recurse "COMSOL_6_3_mcp_skill\skills\comsol-63-operations" $dest
```

Restart opencode. The skill's `description` frontmatter will match COMSOL-related tasks automatically — no explicit invocation needed.

### Option B — Claude Code (project-level)

Drop the repo (or just `CLAUDE.md` + `skills/`) into your project. Claude Code reads `CLAUDE.md` from the project root on startup, and the `@skills/comsol-63-operations/SKILL.md` import pulls the skill body into context.

For a personal (global) install, copy `CLAUDE.md`'s import line into `~/.claude/CLAUDE.md`:

```markdown
@/absolute/path/to/COMSOL_6_3_mcp_skill/skills/comsol-63-operations/SKILL.md
```

### Option C — Codex CLI / Gemini CLI / Cursor (AGENTS.md)

These tools read `AGENTS.md` from the project root automatically. Clone the repo into (or beside) the project you're working in, and `AGENTS.md` will tell the agent to open `skills/comsol-63-operations/SKILL.md` whenever a COMSOL task comes up.

For a global Codex install, append a pointer line to `~/.codex/AGENTS.md`:

```markdown
For COMSOL 6.3 MCP tasks, read /absolute/path/to/COMSOL_6_3_mcp_skill/skills/comsol-63-operations/SKILL.md first.
```

### Option D — read it directly

The skill is a single markdown file. Open `skills/comsol-63-operations/SKILL.md` and use it as a reference when writing COMSOL MCP code or driving the server by hand.

## Prerequisites

The skill assumes you already have:

- COMSOL Multiphysics 6.3 installed and running
- The [comsol MCP server fork](https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_3_Calibrated) configured in your AI tool (opencode / Claude Code / Codex)
- MPh 1.3.1 + JPype in the MCP server's Python environment

The skill itself has no dependencies — it's just instructions for the agent.

## Repository layout

```
.
├── skills/
│   └── comsol-63-operations/
│       └── SKILL.md          # single source of truth (markdown + frontmatter)
├── CLAUDE.md                 # Claude Code entry: @import the skill
├── AGENTS.md                 # Codex / Gemini CLI / Cursor entry: pointer
├── README.md                 # this file (English)
├── README_CN.md              # 中文
└── LICENSE                   # MIT
```

## Provenance

The skill was distilled from a real debugging session where an AI assistant (opencode + glm-5.2) calibrated the MCP server's `src/tools/` for 6.3 clientapi under human direction, then verified end-to-end through the MCP tool interface. The companion fork repo has the code changes and full `git log`.

## License

MIT — see [LICENSE](LICENSE).

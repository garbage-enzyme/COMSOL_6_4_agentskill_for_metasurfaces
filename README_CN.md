# COMSOL MCP Skill — 6.4+ ClientAPI 操作技能

[English](README.md) | 中文

一个 agent skill，教 AI 编码助手如何通过 [comsol MCP server](https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated) 及 MPh 1.3.1 standalone / `clientapi` 驱动 **COMSOL Multiphysics 6.4+**，以及在 `src/tools/` 里写/改代码时如何应对 API 不匹配。

## 兼容：opencode、Claude Code、Codex（以及任何读 AGENTS.md 的工具）

本仓库遵循开放的 agent-skill 约定，**同一份** skill 内容可在三个 AI 编码工具间复用，无重复：

| 工具 | 它读的入口文件 | skill 如何加载 |
| --- | --- | --- |
| **opencode** | `skills/comsol-64-metasurface/SKILL.md` | 从 `~/.config/opencode/skills/` 自动加载，任务匹配 frontmatter `description` 时触发 |
| **Claude Code** | `CLAUDE.md`（项目根） | `CLAUDE.md` 内含 `@skills/comsol-64-metasurface/SKILL.md`，import 语法把完整 skill 注入上下文 |
| **Codex CLI**（及任何读 AGENTS.md 的工具：Gemini CLI、Cursor） | `AGENTS.md`（项目根） | `AGENTS.md` 被自动读取，其中指示 agent 在 COMSOL 任务时打开 `skills/comsol-64-metasurface/SKILL.md` |

唯一的内容源是 `skills/comsol-64-metasurface/SKILL.md`（markdown + YAML frontmatter）。`CLAUDE.md` 和 `AGENTS.md` 是薄指针 —— 无内容重复。

## Skill 覆盖内容

- **clientapi vs 直接 Model 的差异** —— `tags()` 遍历 vs int 索引、`feature().size()` vs `len()`、`getNBoundaries()` 首字母大写、`physics().create(tag, type, sdim_String)` 三参数、study step type 用完整名、`mesh.getNumElem()`、用 `jm.study().run()` 而非 `m.study().run()`。
- **6.3+/6.4 Electrostatics 陷阱** —— 默认 `fsp1` (FreeSpace) domain feature 用真空 ε₀，忽略材料 `relpermittivity`，必须加 `ChargeConservation` feature + 材料节点。Block 边界编号不是 1–6 ↔ −x/+x/−y/+y/−z/+z（bnd 3 = z=0 面，bnd 4 = z=最大值面）。`Terminal` 的 `V0` 不能正确约束电压 —— 用 `ElectricPotential`。表达式语法：`1[V]^2` 会报语法错误，必须 `(1[V])^2`。
- **网格与求解陷阱** —— COMSOL 不自动创建 mesh 序列；study step type 必须用完整名（`Stationary`，不是 `stat`）。
- **电容验证 recipe** —— 纯 Python（mph Client）和 MCP 工具流程两版平行板电容器端到端 recipe。验证结果：**C = 1.8593794414 pF**，理论值 1.8593794407 pF（误差 4 × 10⁻¹⁰ pF）。
- **Wave Optics 超表面陷阱** —— `PeriodicStructure`、Layered Drude 边界、波长-参数扫描（`plistarr`）、PML/periodic-port 陷阱，以及 MIM emitter 的 mesh/mode-selection 诊断。
- **调试技巧** —— JPype 反射探测 clientapi 重载、用 `Box` selection 按坐标识别边界编号、`m.evaluate('V')` 取场数组。

## 安装

### 方式 A —— opencode

opencode 从 `~/.config/opencode/skills/`（Windows 下 `%USERPROFILE%\.config\opencode\skills\`）自动加载 skill。把 skill 文件夹拷过去：

```bash
git clone https://github.com/garbage-enzyme/COMSOL_6_4_mcp_skill.git
mkdir -p ~/.config/opencode/skills
cp -r COMSOL_6_4_mcp_skill/skills/comsol-64-metasurface ~/.config/opencode/skills/
```

Windows PowerShell：

```powershell
git clone https://github.com/garbage-enzyme/COMSOL_6_4_mcp_skill.git
$dest = "$env:USERPROFILE\.config\opencode\skills"
New-Item -ItemType Directory -Path $dest -Force | Out-Null
Copy-Item -Recurse "COMSOL_6_4_mcp_skill\skills\comsol-64-metasurface" $dest
```

重启 opencode。skill 的 `description` frontmatter 会自动匹配 COMSOL 相关任务 —— 无需显式调用。

### 方式 B —— Claude Code（项目级）

把仓库（或只 `CLAUDE.md` + `skills/`）放进你的项目。Claude Code 启动时从项目根读 `CLAUDE.md`，其中的 `@skills/comsol-64-metasurface/SKILL.md` import 会把 skill 内容注入上下文。

全局安装：把 `CLAUDE.md` 的 import 行拷进 `~/.claude/CLAUDE.md`：

```markdown
@/absolute/path/to/COMSOL_6_4_mcp_skill/skills/comsol-64-metasurface/SKILL.md
```

### 方式 C —— Codex CLI / Gemini CLI / Cursor（AGENTS.md）

这些工具会自动读项目根的 `AGENTS.md`。把仓库 clone 进（或旁边）你正在做的项目，`AGENTS.md` 会在 COMSOL 任务出现时指示 agent 打开 `skills/comsol-64-metasurface/SKILL.md`。

Codex 全局安装：在 `~/.codex/AGENTS.md` 追加一行指针：

```markdown
For COMSOL 6.4+ MCP tasks, read /absolute/path/to/COMSOL_6_4_mcp_skill/skills/comsol-64-metasurface/SKILL.md first.
```

### 方式 D —— 直接看

skill 就是单个 markdown 文件。打开 `skills/comsol-64-metasurface/SKILL.md`，当写 COMSOL MCP 代码或手动驱动 server 时当参考用。

## 前提条件

本 skill 假设你已有：

- 已安装并运行的 COMSOL Multiphysics 6.4 或更新版本
- 在你的 AI 工具（opencode / Claude Code / Codex）里配置好的 [comsol MCP server fork](https://github.com/garbage-enzyme/COMSOL_Multiphysics_MCP_6_4_Calibrated)
- MCP server 的 Python 环境里有 MPh 1.3.1 + JPype

skill 本身无依赖 —— 它只是给 agent 的指令。

## 仓库结构

```
.
├── skills/
│   └── comsol-64-metasurface/
│       └── SKILL.md          # 唯一内容源（markdown + frontmatter）
├── CLAUDE.md                 # Claude Code 入口：@import skill
├── AGENTS.md                 # Codex / Gemini CLI / Cursor 入口：指针
├── README.md                 # 本文件（英文）
├── README_CN.md              # 中文
└── LICENSE                   # MIT
```

## 来源

本 skill 提炼自真实调试会话：AI 助手在人工指导下为 COMSOL standalone `clientapi` 校准 MCP server 的 `src/tools/`，随后通过 MCP 工具接口端到端验证。配套的 fork 仓库有代码改动和完整 `git log`。

## 许可证

MIT —— 见 [LICENSE](LICENSE)。

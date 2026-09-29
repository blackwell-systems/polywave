[English](../../README.md) · **简体中文** · [Русский](README.ru.md) · [हिन्दी](README.hi.md) · [العربية](README.ar.md)

# Polywave

<p align="center">
  <img src="../../assets/logo.png" alt="Polywave" width="600" />
</p>

<p align="center">
  <a href="https://github.com/blackwell-systems"><img src="https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg" alt="Blackwell Systems" /></a>
  <img src="https://img.shields.io/badge/version-0.11.0-blue" alt="Version" />
  <a href="https://agentskills.io"><img src="../../assets/badge-agentskills.svg" alt="Agent Skills" /></a>
  <a href="https://buymeacoffee.com/blackwellsystems"><img src="https://img.shields.io/badge/buy%20me%20a%20coffee-donate-yellow.svg" alt="Buy Me A Coffee" /></a>
</p>

**并行运行的 AI 智能体互不破坏彼此的代码，就在你已经在用的 CLI 里。**

Polywave 是一个轻量级的叠加层，而不是一个智能体平台。你继续在 Claude Code（或 Codex）中工作。安装一次，之后你的全部交互界面就是你已在使用的工具里的 `/polywave`：每个智能体都拥有自己的 worktree，每个文件都被指派给恰好一个智能体，而且在任何智能体动你的代码之前，你就能看到完整的计划。冲突在规划阶段解决，而不是在合并阶段解决。

你无需采用某个运行时、迁移到新工具，也不用运行消息传递／记忆／编排引擎。安装过程会添加一个 skill、一组 hook，以及 `polywave-tools` 二进制文件。skill 和 hook 驱动整个工作流：它们在底层调用该二进制文件，因此在正常使用中你只需输入 `/polywave scout "feature"` 和 `/polywave wave`。当你想用时，CLI 随时可用（用于恢复、脚本编写、CI、进阶用法），但大多数会话都不会直接接触它。重量级智能体框架要求你搬进它们的世界才能获得并行能力；Polywave 则在你自己的世界里与你相遇，并让合并变得安全。

> 以 [Agent Skill](https://agentskills.io)（开放标准）形式发布。兼容 Claude Code、Cursor、GitHub Copilot 以及其他兼容 Agent Skills 的工具。

> **初次接触 Polywave？**
> 1. 阅读本 README（15 分钟）
> 2. 阅读 [QUICKSTART.md](../../implementations/claude-code/QUICKSTART.md)（20 分钟），了解一个完整示例
> 3. 试用一下：在一个测试项目上运行 `/polywave scout "feature"`
> 4. 深入了解：[polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) 查看完整规范

## 为什么

你以前跑过并行智能体。你知道会发生什么：两个智能体编辑同一个文件，合并产出一堆垃圾，你花在修复上的时间比按顺序做完还要长。或者更糟：合并悄无声息地成功了，因为两个智能体改的是同一文件中的不同函数，但它们对共享状态做出了相互矛盾的假设。你直到运行时才发现问题。

大多数框架试图用更好的提示词来解决这个问题。Polywave 用结构来解决它：

- **文件所有权互不相交。** Scout 在写下任何代码之前，就把每个文件指派给恰好一个智能体。同一个 wave 里的两个智能体不可能对同一个文件产生编辑。这一指派在工具边界处强制执行，而不是交由智能体的自觉；违规变得不可能发生。合并冲突从结构上被消除。
- **每个智能体的 worktree 隔离。** 每个智能体在它自己的 git worktree 中工作，那是一个拥有独立文件树的单独目录。并发的构建、测试和工具缓存写入不会在共享状态上发生竞争。
- **执行前的人工评审。** 你会看到完整的计划（文件指派、接口契约、wave 结构）并在任何智能体启动之前批准它。这是改动架构成本最低的最后一个节点。
- **适用性闸门。** 当工作无法干净地分解时，Polywave 会说“不行”。一次“不适合”的评估可以防止糟糕的分解方案演变成代价高昂的失败。

Polywave 不是一个智能体运行时。它不路由任务、不管理智能体间的消息传递，也不维护跨会话的记忆。它是一个协调协议：安全地划分工作、验证划分、独立地启动智能体、确定性地合并。智能体并行运行但彼此不通信；正确性来自划分，而非来自协作。

这是一个刻意选择的“重量级别”。完整的智能体引擎（Hermes、各类 swarm 框架之流）把执行运行时、记忆、消息传递和编排捆绑成一个你需要采用的平台。Polywave 一概不带这些。它搭载在你已有的智能体运行时之上，只额外加了那些平台所缺失的一样东西：让并行编辑变得安全的文件所有权划分与确定性合并。如果你想要一整套智能体引擎，那就去用一套。如果你想让你现有的 CLI 同时跑好几个编码智能体而不让合并爆掉，那正是 Polywave 的用武之地。

系统有七种[参与者角色](https://github.com/blackwell-systems/polywave-protocol/blob/main/participants.md)，但你只与其中两种交互：**Orchestrator**（你的 Claude Code 会话，负责协调一切）和 **Scout**（分析代码库、指派文件、写下计划）。其余五种（Scaffold Agent、Wave Agents、Integration Agent、Critic Agent、Planner）会在需要时自动运行。

## 如何运作

**运行 Polywave 时会发生什么：**

1. 你运行 `/polywave scout "feature"` -> Scout 分析代码库，把文件指派给各个智能体
2. Scout 写出 IMPL 文档（一份 YAML 协调工件，定义文件所有权、接口契约和 wave 结构）-> 你评审 wave 结构
3. 你运行 `/polywave wave` -> 如有需要，Scaffold Agent 创建脚手架文件
4. Wave Agents 并行启动 -> 每个在隔离的 worktree 中处理互不相交的文件
5. Orchestrator 合并 -> 运行测试 -> 清理 worktree

**关键机制：**

- **Orchestrator：** 你会话中的同步协调智能体。启动智能体、强制执行文件所有权、执行合并流程、运行验证闸门。

- **Scout：** 异步智能体。分析代码库，产出带有依赖图、接口契约、文件所有权表和 wave 结构的 IMPL 文档。每个文件都被指派给恰好一个智能体。

- **Scaffold Agent：** 若需要共享类型，则在 Wave 1 之前运行一次。根据 IMPL 文档的契约创建共享类型文件，验证编译，并提交到 HEAD。

- **Wave Agents：** 并行运行的异步智能体。每个拥有互不相交的文件，针对冻结的接口契约进行实现，运行验证闸门，提交工作，并写出完成报告。

- **Integration Agent：** 在 wave 合并之后运行。把 wave 智能体产生的新导出接线到调用方代码中。非致命；若接线失败，则把缺口报告给人类。

协议内建了一个**适用性闸门**，在产出任何智能体提示词之前会先回答[五个问题](https://github.com/blackwell-systems/polywave-protocol/blob/main/preconditions.md)。如果前置条件不成立，scout 会发出 NOT SUITABLE 并停止。

<picture>
  <source media="(prefers-color-scheme: dark)" srcset="assets/diagrams/polywave-scout-wave-dark.svg">
  <img src="../../assets/diagrams/polywave-scout-wave-light.svg" alt="Polywave scout + wave execution flow">
</picture>

## 快速开始

**前置要求：** Git 2.20+、jq 1.6+、Claude Code。强烈推荐为你的技术栈配一个语言服务器（例如 `gopls`、`rust-analyzer`、`pyright`、`typescript-language-server`）：智能体用 LSP 来导航，没有它则退化为较慢的 grep。你不需要 polywave-protocol 或 polywave-web。

```bash
# 1. Install skill files, hooks, and Agent permission
git clone https://github.com/blackwell-systems/polywave.git ~/code/polywave
~/code/polywave/install.sh    # configures Agent permission, symlinks skills, installs hooks

# 2. Install polywave-tools CLI (Homebrew — recommended)
brew install blackwell-systems/tap/polywave-tools

# 3. Initialize your project
cd your-project
polywave-tools init            # auto-detects language, build, and test commands

# 4. Verify
polywave-tools verify-install  # checks CLI, git, LSP, skill files, hooks, permissions
```

<details>
<summary>Alternative CLI install methods</summary>

**预构建二进制文件**（无需 Go 工具链）：从[最新发布](https://github.com/blackwell-systems/polywave-go/releases/latest)下载，并把它移动到你的 `PATH` 上。

**Go install：**
```bash
go install github.com/blackwell-systems/polywave-go/cmd/polywave-tools@latest
```
注意：`go install` 会把二进制文件放到 `$(go env GOPATH)/bin`（通常是 `~/go/bin`）。如果之后 `polywave-tools` 报告 “command not found”，说明该目录不在你的 `PATH` 上 — 把它加上：
```bash
echo 'export PATH="$(go env GOPATH)/bin:$PATH"' >> ~/.zshrc && source ~/.zshrc
```
以这种方式安装的二进制文件其版本会报告为 `dev`；如需带版本号的构建，请使用 Homebrew 或发布二进制文件。
</details>

**5. 重启 Claude Code**，然后运行你的第一次 scout：

```bash
/polywave scout "add a caching layer to the API client"
```

**子命令：**

| Command | Purpose |
|---------|---------|
| `/polywave scout "<feature>"` | 分析代码库，产出 IMPL 文档 |
| `/polywave wave` | 执行下一个待处理的 wave |
| `/polywave wave --auto` | 无人值守地执行所有剩余的 wave |
| `/polywave auto "<feature>"` | 在一个命令里完成 scout + 确认 + wave |
| `/polywave status` | 显示当前 wave 和智能体进度 |
| `/polywave bootstrap "<project>"` | 从零设计新项目结构 |
| `/polywave interview "<description>"` | 结构化的需求收集 |
| `/polywave program --impl <slug> ...` | 把排队的 IMPL 打包成一个并行程序 |
| `/polywave program plan/execute/status/replan` | 多特性规划与按层级门控的执行 |
| `/polywave amend --add-wave/--redirect-agent/--extend-scope` | 修改进行中的 IMPL |

**第一次使用 Polywave？** 参见 [QUICKSTART.md](../../implementations/claude-code/QUICKSTART.md)，其中有分步指引和示例输出。

## 各代码仓库

| Repository | Purpose |
|-----------|---------|
| [polywave-protocol](https://github.com/blackwell-systems/polywave-protocol) | 协议规范：不变式、执行规则、状态机、消息格式 |
| **polywave**（本仓库） | Claude Code 实现：Agent Skill、hook、提示词、智能体模板 |
| [polywave-go](https://github.com/blackwell-systems/polywave-go) | Go 引擎、Protocol SDK，以及 `polywave-tools` CLI |
| [polywave-web](https://github.com/blackwell-systems/polywave-web) | Web UI 和 HTTP/SSE 服务器 |

## 何时使用

当工作有清晰的文件分界、接口能在实现开始之前就定义好、并且每个智能体承担的工作量足以值得并行运行时，Polywave 就物有所值。构建／测试周期若超过 30 秒，会进一步放大这份收益。

如果工作无法干净地分解，Scout 会明说。它会先运行一道适用性闸门，宁可发出 NOT SUITABLE，也不会强行推进一个糟糕的分解方案。

## 并行安全如何实现

Polywave 强制执行两条相互独立的约束，二者共同使并行执行保持正确：

**文件所有权互不相交**防止合并冲突。每一个将被改动的文件都在 IMPL 文档中被指派给恰好一个智能体。同一个 wave 里的两个智能体不可能对同一个文件产生编辑，因此合并步骤总是无冲突的。

**worktree 隔离**防止执行期的相互干扰。每个智能体在它自己的 git worktree 中工作，那是一个共享同一 git 历史但拥有独立文件树的单独目录。并发的构建、测试和工具缓存写入不会在共享状态上发生竞争。

两条约束谁也替代不了谁。有互不相交的所有权而无 worktree：合并是安全的，但并发构建会不稳定。有 worktree 而无互不相交的所有权：执行是干净的，但合并会产生无法解决的冲突。两者都必须成立。

### worktree 隔离防御（6 层）

智能体并不总是遵守隔离指令。Polywave 把 worktree 隔离当作一个基础设施问题来对待，而非一个协作问题，并以基于 hook 的强制执行（E43）作为主要机制。

| Layer | Mechanism | Type |
|-------|-----------|------|
| **E43** | **基于 hook 的强制执行**：Claude Code 生命周期 hook（SubagentStart、PreToolUse:Bash、PreToolUse:Write/Edit、SubagentStop）自动注入环境变量、在 bash 调用前预置 cd 命令，并在工具边界处拦截越界写入。违规变得不可能发生，而不仅仅是被检测到。 | **预防（主要）** |
| 0 | **预提交 hook**：由 `polywave-tools create-worktrees` 自动安装。在 wave 活动期间阻止对 main 的提交。Orchestrator 通过 `POLYWAVE_ALLOW_MAIN_COMMIT=1` 绕过。 | 预防 |
| 1 | **手动预创建 worktree**：Orchestrator 在任何智能体启动之前创建好所有 worktree | 确定性 |
| 2 | **`isolation: "worktree"` 参数**：每个智能体启动时都在工具层面指定 worktree 隔离 | 工具层面 |
| 3 | **Field 0 自我校验**：智能体通过一次简短检查确认所在分支（主要的强制执行是 `validate_worktree_isolation` SubagentStart hook） | 协作性 |
| 4 | **合并时的绊线**：Orchestrator 在合并前统计每个 worktree 分支上的提交数。零提交 = 隔离失败。停止并给出恢复选项。 | 确定性 |

## Polywave-Teams（实验性）

[`docs/proposals/polywave-teams/`](../../docs/proposals/polywave-teams/) 是一个使用 Claude Code Agent Teams 的备选执行层。相同的协议、相同的 IMPL 文档、相同的 Scout。不同的是 wave 的底层管道：队友取代后台的 Agent 工具调用，提供智能体间的消息传递和实时的偏离告警。

## 博客文章

围绕这一模式、从吃自家狗粮中汲取的经验教训，以及协议如何演进的四部曲系列：

1. [Polywave: A Coordination Pattern for Parallel AI Agents](https://blog.blackwell-systems.com/posts/scout-and-wave/)。这一模式：朴素并行的失败模式、scout 交付物、wave 执行，以及来自 brewprune 的一个完整示例。
2. [Polywave, Part 2: What Dogfooding Taught Us](https://blog.blackwell-systems.com/posts/scout-and-wave-part2/)。审计－修复－审计的循环、开销测量（被忽视时慢 88%）、Quick 模式，以及新项目的引导（bootstrap）问题。
3. [Polywave, Part 3: Five Failures, Five Fixes](https://blog.blackwell-systems.com/posts/scout-and-wave-part3/)。skill 文件如何从 400 行的单体拆分开来、为什么版本头很重要，以及由真实失败驱动的五处 scout 提示词修复。
4. [Polywave, Part 4: Trust Is Structural](https://blog.blackwell-systems.com/posts/scout-and-wave-part4/)。Scaffold Agent、5 层 worktree 隔离防御，以及为什么正确性属于基础设施而非协作。

## 许可证

[MIT OR Apache-2.0](../../LICENSE)

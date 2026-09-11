# Local-First Server Development

[English](README.md)

`local-first-server-dev` 是一个面向环境分离型工程任务的 Agent Skill：代码可以在本地开发工作区中阅读、修改并完成部分验证，但权威编译、仿真、训练、硬件访问或运行时验证必须由用户在受控服务器或目标设备上执行。

它的目标是：

> 把只有目标环境才能完成的事情留给目标环境，把能够提前完成的复杂工程工作全部留在本地。

## 为什么需要这个 Skill

GPU 服务器、HPC 集群、机器人工作站、嵌入式设备、专有 SDK 和生产类 Linux 环境通常是最终验证所必需的，但并不适合让 AI 在其中继续交互式开发。如果没有明确工作流，Coding Agent 可能会在本地过度复刻服务器环境，也可能把尚未准备好的问题交给用户到服务器现场调试。

本 Skill 建立以下闭环：

```text
本地分析
    -> 本地修改
    -> 本地验证
    -> Human Gate
    -> 用户在受限目标环境执行
    -> 返回日志和结果
    -> 继续本地迭代
```

服务器或目标设备应主要承担 `apply`、`build`、`run` 和 `collect`，而不是源码探索、方案设计或现场修改代码。

## 核心行为

- **Local First：**优先在本地仓库完成分析、实现、测试、配置、Mock 和诊断准备。
- **Evidence, Not Assumption：**区分实际执行、静态审查、本地未执行和目标环境待验证。
- **Human-Gated Target：**不连接或操作受限服务器、目标设备、真实硬件和生产环境。
- **Minimize Target Burden：**为用户准备最短的同步、Preflight、正式运行和失败信息收集流程。

如果当前工作区本身位于 Remote SSH、远程挂载目录或受限服务器上，Skill 将停止写入和执行。边界由操作最终作用的位置决定，而不是路径看起来是否像本地路径。

## 适用场景

- CUDA 或 GPU 训练项目；
- Isaac Lab、Isaac Sim、MJLab、MuJoCo 等仿真项目；
- ROS 2 和机器人部署；
- Linux 专用程序或交叉编译工程；
- 嵌入式目标、传感器、CAN、相机和专有 SDK；
- Slurm/HPC 作业及其他受控计算环境。

仅仅出现 PyTorch、Linux、CUDA、ROS 或 MuJoCo 等关键词并不足以触发本 Skill。只有本地开发环境与权威运行环境确实分离时，才需要这一工作流。

## 不做什么

- 不通过 SSH、远程挂载、SCP 或 rsync 访问和修改受限目标；
- 不启动或停止远程构建、训练、仿真、部署和硬件进程；
- 不为了宣称完整验证而在本地重建整套服务器环境；
- 不要求用户配置 orchestrator、reviewer、tester 或 handoff agent；
- 不提供针对某一技术栈的完整环境搭建手册。

普通 Git 托管操作服从宿主 Agent 权限和用户授权。但如果某次 push、merge 或 release 会触发受限服务器部署、训练、生产发布或硬件执行，它就属于被 Human Gate 保护的执行链，必须留给用户操作。

## 安装

本仓库的 [`SKILL.md`](SKILL.md) 是唯一行为规范来源。仓库采用开放的 Agent Skill 目录形态，但只有经过实际测试的平台才会被声明为兼容。

### Codex

用户级安装可以把[本仓库](https://github.com/Mingyang-Sheep/local-first-server-dev)克隆到 Codex 用户 Skill 目录。

PowerShell：

```powershell
New-Item -ItemType Directory -Force "$HOME\.agents\skills" | Out-Null
git clone https://github.com/Mingyang-Sheep/local-first-server-dev.git "$HOME\.agents\skills\local-first-server-dev"
```

POSIX shell：

```bash
mkdir -p "$HOME/.agents/skills"
git clone https://github.com/Mingyang-Sheep/local-first-server-dev.git "$HOME/.agents/skills/local-first-server-dev"
```

仓库级安装则将 Skill 文件夹放在目标仓库的 `.agents/skills/` 下。Codex 支持通过 `$local-first-server-dev` 显式调用，也可能根据 description 自动选择。当前路径和调用规则以 [OpenAI 官方 Skill 文档](https://developers.openai.com/codex/skills)为准。

其他 Coding Agent 应按照各自最新官方方式加载 `SKILL.md`、Rules 或项目指令。本仓库采用平台中立写法，不代表未经测试的平台已经受到支持。

## 兼容性状态

| 宿主平台 | 状态 |
| --- | --- |
| Codex | 已在本地完成格式校验和行为冒烟测试 |

在实际测试之前，不对其他宿主平台作支持声明。只有真实使用证明有必要时，才增加轻量平台包装；核心行为仍只维护一份。

## 使用方式

可以明确调用：

```text
使用 $local-first-server-dev 在本地完成这项修改。最终构建和运行由我在 GPU 服务器上执行。
```

也可以直接说明环境边界：

```text
本地没有机器人 SDK。请把这里能够可靠完成的修改和验证全部做完，然后给我最短的目标设备 Preflight 和运行步骤。
```

在任务完成或准备目标环境交接时，Agent 会说明本地完成内容、实际验证证据、剩余目标验证、人工执行命令、成功标准以及失败后需要返回的脱敏信息。普通讨论不会机械套用这份交付格式。

## 示例

- [GPU 训练或仿真](examples/gpu-or-simulation.md)
- [目标设备或交叉编译](examples/target-device-or-cross-compile.md)
- [服务器反馈闭环](examples/feedback-loop.md)

示例中的命令会明确标记为模板。真实任务必须从当前项目中获取命令，并只询问确实缺失的必要信息。

## 仓库结构

```text
local-first-server-dev/
|-- SKILL.md
|-- README.md
|-- README.zh-CN.md
|-- LICENSE
`-- examples/
    |-- gpu-or-simulation.md
    |-- target-device-or-cross-compile.md
    `-- feedback-loop.md
```

首版刻意不加入远程自动化、服务器脚本、平台适配目录或技术栈手册。只有真实使用暴露出需要时，才增加行为评测或轻量适配。

## 贡献原则

改动应聚焦于跨项目都有价值的工作流决策。不要在平台包装中复制行为规范，也不要把某个项目的环境细节升级为通用规则。提交兼容性结论时，应同时记录宿主平台、调用方式、测试场景和实际结果。

## 许可证

[MIT](LICENSE)

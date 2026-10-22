# Code Study

A reusable Codex Skill: **map the code → run a minimal example → trace and debug**.

将开源代码学习变成可亲自验证的过程。适用于模型、机器人、训练/推理框架和普通软件；
不绑定编程语言、IDE 或硬件。解释默认使用学习者的语言。

## 目录与安装

```text
.
├── .agents/skills/code-study -> ../../skills/code-study
├── skills/code-study/
│   ├── SKILL.md                 # 完整技能指令，唯一源文件
│   └── agents/openai.yaml       # 展示名称和默认调用提示
├── docs/smoke-test.md            # 实际示例、运行结果、断点路线及验收记录
├── LICENSE                      # MIT
├── README.md
└── TASK.md                      # 原始需求
```

[完整 SKILL.md](skills/code-study/SKILL.md) 只有三阶段流程和必要约束，
不加载仓库说明或测试记录，也不要求额外脚本和运行依赖。

按照已查阅的 [Codex 官方 Skills 文档](https://developers.openai.com/codex/skills/)，
项目技能放在 `.agents/skills`，个人技能放在 `~/.agents/skills`；支持目录符号链接。
本仓库用链接指向唯一源目录，避免维护两份指令。

- **在本仓库使用：** 克隆后从仓库中启动 Codex，使用下方调用示例。
- **给其他项目安装：** 将 `skills/code-study` 整个目录复制到目标项目的
  `.agents/skills/code-study`，或复制到 `~/.agents/skills/code-study` 供个人全局使用。
  已存在时先检查内容，避免覆盖本地定制。
- Codex 会自动发现变更；若技能未出现，重启会话。旧版客户端请核对其支持的发现路径。

例如，从本仓库根目录安装到尚未存在该技能的个人目录：

```bash
mkdir -p ~/.agents/skills
test ! -e ~/.agents/skills/code-study && test ! -L ~/.agents/skills/code-study && \
  cp -R skills/code-study ~/.agents/skills/code-study
```

## 调用与交互

```text
$code-study 帮我了解这个仓库，先给一屏代码地图和最小运行入口。
$code-study 我已跑通官方 Demo，跳过前两阶段，带我追踪输入到输出。
$code-study Walk me through the Quick Start, then suggest debugger stops.
```

自动匹配默认开启：学习陌生项目、理解核心机制、沿示例探索代码时适用。
没有学习目标的普通修 Bug、功能实现或代码审查不应触发。

| 阶段 | 交付 | 交互方式 |
| --- | --- | --- |
| 1：建立地图 | 简短架构概览、约 3–7 个关键文件或符号、阅读起点和可略过部分 | 初次概览后由你选择深入方向 |
| 2：最小运行 | 依赖与成本检查、实际命令、Python `launch.json`、输入输出和运行证据 | 大权重优先 weight-free，复用已有运行 |
| 3：沿路径调试 | 一段数据/调用流、1–3 个断点、变量和预期现象、可验证问题 | 你亲自观察，再逐段继续；不假定能控制 IDE |

明确要求三个阶段时连续推进；已经熟悉或已经跑通的部分可以跳过。
初始学习笔记默认约 500–800 字、1–2 屏，架构概览尽量不超过 300 字；
先用口语化的比喻、形象图例和实例讲清直觉，再定位代码，按当前问题补充技术细节。
明确区分运行验证、代码确认和待验证推断。

## 三个学习偏好

- **大权重先拆开学：** 以参数量 **≥5B（50 亿）** 为大权重门槛；其他下载或带权重
  复现麻烦的项目也优先考虑 weight-free。像先拆开看看流水线的传送带和分拣器：
  运行真实的预处理、张量拼装或小型核心运算，明确跳过权重加载和依赖权重的环节。
  这能验证所选代码片段，模型的预测能力仍需带权重验证；不默认随机初始化整个大模型。
- **环境配好就能按 F5：** 构建 Python 学习环境时，实际生成或合并目标项目的
  `.vscode/launch.json`，配置 `debugpy`、解释器、入口、参数和工作目录。
  选了 weight-free 路径，就调试该路径。保留已有配置，并说明调试器是否实际运行过。
- **讲得像面对面聊天：** 用熟悉的比喻配合方框、箭头、网格或 Mermaid 图解释。
  比如把数据传递画成包裹过站，但仍标清真实函数、数据和比喻不适用的地方。
  术语按需出现，shape、代码位置和证据仍须精确。

## 验证与发布

[实际验收记录](docs/smoke-test.md) 使用本机开源 CPython `json.tool`，
包含成功/错误输入的真实输出及 `pdb` 命中的三个断点。无需克隆项目或下载数据。
本次为读取 Skill 后的人工应用测试；自动匹配仅做描述级审阅。

若本机已有 bundled `skill-creator` 及带 PyYAML 的 Python，可运行其结构检查器：

```bash
python /path/to/skill-creator/scripts/quick_validate.py skills/code-study
```

发布时分发 `skills/code-study/` 并保留 [MIT 许可](LICENSE) 即可。
示例只引用 CPython 的本机源码位置，没有复制其实现；CPython 遵循自己的许可。

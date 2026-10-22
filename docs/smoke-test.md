# Code Study 实际验收

2026-10-21，读取 `skills/code-study/SKILL.md` 后，在本机 CPython 3.12.3
现有标准库上应用三个阶段。没有下载项目、权重或数据；没有改动示例实现。
以下先给学习笔记，复现细节和设计检查按需展开。

## 学习笔记

**它是什么？** 把 `json.tool` 想成一个拆包、装包的工作台：
`load/loads` 拆开 JSON 文本，得到 Python 字典和列表；`dump` 再按选项把它们装回文本。
命令行层像接单员，决定从哪读、往哪写、要不要排序。这个比喻只对应读写转换，
实际操作发生在内存里。

```mermaid
flowchart LR
    A["JSON 文本：待拆的包裹"] -->|load / loads| B["Python 对象：取出的物品"]
    B -->|dump / iterencode| C["JSON 文本：按规则装回"]
```

**先看例子。** 【运行验证】输入 `{"b": [2, 1], "a": true}`，
输出 `{"a":true,"b":[2,1]}`：对象字段排序、空白压缩，列表顺序保持不变。
无第三方依赖、GPU、权重或服务。任意目录可执行：

```bash
printf '%s\n' '{"b": [2, 1], "a": true}' | /usr/bin/python3 -m json.tool --sort-keys --compact
```

**关键代码地图。** 本机源码基目录为 `/usr/lib/python3.12/json/`，
以下位置已读源码确认；其他版本先重新定位符号。

| 位置 | 作用 |
| --- | --- |
| `tool.py:19 main` | 首先阅读：选项、输入输出及错误处理 |
| `__init__.py:274 load` / `:299 loads` / `:120 dump` | 文件 → 文本 → 对象，以及写回文本 |
| `decoder.py:332 JSONDecoder.decode` | 解析及检查剩余文本 |
| `encoder.py:205 JSONEncoder.iterencode` | 将对象变成逐段文本；当前不必展开底层加速器 |

**下一步调试。** 【运行验证】用同一输入文件，`pdb` 已命中以下三处；
VS Code 或其他调试器也可手动设断点，不要求 Agent 操控 IDE。

| 断点（相对上述基目录） | 观察与验证 |
| --- | --- |
| `tool.py:62 main` | `dump_args`：`sort_keys=True`、`indent=None`、`separators=(',', ':')`；不同则先核对启动参数 |
| `decoder.py:338 decode` | `s` 为文本，`obj` 为字典，`a` 为布尔值；栈中有 `main → load → loads → decode`；不同则核对输入和分支 |
| `tool.py:75 main` | `obj` 仍先 `b` 后 `a`；步入 `json.dump`，观察参数如何进入输出；若顺序已改变，回查解析配置 |

先停在第二处亲自检查：**字段排序发生在解析时，还是输出时？**
比较两个对象断点和最终文本，再决定是否继续追进编码器。
当前示例覆盖普通 JSON 解析/输出；`--json-lines` 和机器人/模型运行尚未测试。

<details>
<summary>复现步骤与实际结果</summary>

解释器为 `/usr/bin/python3`（3.12.3），加载的 `json` 为
`/usr/lib/python3.12/json/__init__.py`。固定解释器是为了匹配已读源码；
本机默认 `python` 来自 Conda，不能假定路径相同。

有效输入命令见学习笔记。实际退出码 `0`，标准输出为
`{"a":true,"b":[2,1]}\n`，标准错误为空。

另运行官方模块文档中的错误示例，验证错误处理：

```bash
printf '%s\n' '{ 1.2:3.4}' | /usr/bin/python3 -m json.tool --sort-keys --compact
```

实际退出码 `1`，标准输出为空，标准错误为：

```text
Expecting property name enclosed in double quotes: line 1 column 3 (char 2)
```

调试时用文件传入 JSON，留出标准输入给 `pdb`；不会改变解析和输出路径：

```bash
sample_dir=$(mktemp -d /tmp/code-study-debug.XXXXXX)
printf '%s\n' '{"b": [2, 1], "a": true}' > "$sample_dir/input.json"
/usr/bin/python3 -m pdb -m json.tool --sort-keys --compact "$sample_dir/input.json"
```

在 `(Pdb)` 提示符逐行执行：

```text
b /usr/lib/python3.12/json/tool.py:62
b /usr/lib/python3.12/json/decoder.py:338
b /usr/lib/python3.12/json/tool.py:75
c
p (options.sort_keys, options.compact, dump_args)
c
p (type(s).__name__, type(obj).__name__, obj, end)
where
c
p (obj, dump_args)
s
where
q
```

本次同序列实际观察摘要（成功退出，`q` 在输出函数入口结束调试）：

```text
(True, True, {'sort_keys': True, 'indent': None, 'ensure_ascii': True, 'separators': (',', ':')})
('str', 'dict', {'b': [2, 1], 'a': True}, 24)
main [tool.py:67] → load [__init__.py:293] → loads [__init__.py:346] → decode [decoder.py:338]
({'b': [2, 1], 'a': True}, {'sort_keys': True, 'indent': None, 'ensure_ascii': True, 'separators': (',', ':')})
step into: /usr/lib/python3.12/json/__init__.py:120 dump
```

这证明断点可达、变量及 Python 调用链匹配该命令。未据此宣称观察到了所有编码器内部细节。
临时文件保留在 `/tmp`，未进入仓库。

</details>

<details>
<summary>设计复核及验收边界</summary>

- 触发描述审阅：“帮我学习陌生仓库”匹配；“已跑通示例，追踪配置”匹配并跳阶段；
  “修复这个 Bug”或“评审 PR”没有学习目标，不匹配。这是描述级人工审阅，
  不是模型自动选择的统计测试。
- 修订：代码地图数量是随规模调整的建议；初次概览与明确要求全流程分别处理，
  防止机械停顿；沿已有运行继续，禁止把未运行路线标成运行事实。
- Skill 仅一个自包含指令文件及 UI 元数据；没有模板、自动安装器或固定语言工具链。
  三阶段各有目标和约束，内容不依赖此 Python 示例；Python 环境按偏好生成调试配置。
- 模型的 Tensor/网络/训练、机器人的消息/状态/时序、普通软件的调用/外部依赖均有
  差异化观察指引。实际执行验收只覆盖本例，不能据此宣称跨领域均已运行测试。
- 指令语言为英文，要求按学习者语言回答；学习笔记篇幅与结构是默认指导，
  不用匹配标题或字数的脆弱测试来替代实际可读性检查。
- 结构检查使用 bundled `skill-creator/scripts/quick_validate.py skills/code-study`，
  本次输出 `Skill is valid!`；它验证元数据和脚手架，学习路线另用上面的实际运行核验。
- 当前会话直接读取 Skill 并应用，没有将已有会话的手工加载说成新会话自动发现测试。
  仓库的 `.agents/skills/code-study` 链接遵循已读取的官方发现规范。

</details>

<details>
<summary>2026-10-22：weight-free、Python 调试配置与图例偏好复核</summary>

本次补充的参数门槛和默认路线做了描述级审阅：

| 场景 | 默认选择 |
| --- | --- |
| 参数量恰好 5B 或大于 5B | 优先运行无需权重的真实代码片段，明确绕过权重依赖 |
| 4B，但权重下载或带权重运行麻烦 | 同样优先 weight-free |
| 参数量未知 | 先查文档或配置，不为计数下载权重或实例化大模型 |
| 用户明确要求验证预训练输出 | 说明必需的权重与成本；weight-free 只能先验证局部机制 |

这些是行为规则审阅，没有宣称跑过大模型；本次未下载或初始化任何模型。

在临时学习工程 `/tmp/code-study-launch-c7s23ap6` 实际创建了
`.vscode/launch.json`，添加下面的配置，同时保留原有 `Existing route` 条目。
示例输入文件 `input.json` 内容与上文相同。

```json
{
  "name": "Code Study: json.tool",
  "type": "debugpy",
  "request": "launch",
  "python": "/usr/bin/python3",
  "module": "json.tool",
  "args": ["--sort-keys", "--compact", "${workspaceFolder}/input.json"],
  "cwd": "${workspaceFolder}",
  "console": "integratedTerminal",
  "justMyCode": false
}
```

实测结果：JSON 可解析，原有条目保留；所用字段在本机 Python Debugger 扩展的
启动配置中受支持。按配置生成的命令返回 `0` 和 `{"a":true,"b":[2,1]}`；
通过扩展自带的 `debugpy` 启动同一路径也返回相同结果。
没有操控 VS Code UI，因此 F5 会话尚未实测。

用户可在 VS Code 中打开这个临时工程，进入“运行和调试”，选择
`Code Study: json.tool`，按前面的三处位置打断点，再按 F5。
临时目录用于本机验证；换机器时由 Skill 按实际工程重新生成配置。

学习笔记已用拆包/装包的比喻与 Mermaid 图表示数据流，并逐一对应真实函数。
术语与准确的输入输出仍保留在对应代码处。

</details>

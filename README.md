# Continuous Copilot

Continuous Copilot 是一个面向 Copilot Agent / CLI 长流程研发任务的配置包。它通过自定义智能体、全局规则、MCP 服务和 hook 约束，让 Copilot 在一次请求中更稳定地完成“探索、规划、执行、验证、状态同步、继续推进”的循环。

另外说明：本插件为GitHub Copilot插件直接导出，未进行测试，hooks原方案是使用外部文件脚本的，现在嵌入了json也未进行测试，由于这是个人配置，MCP使用习惯为个人习惯，若需修改请自行在continuous/agents/Continuous.agent.md中修改。

工具配置部分说明：在本人的使用过程中，会频繁出现网络错误（Claude Opus4.6），发现大部分是出现在创建/修改文件的时候，所以在agent中关闭了自带的edit工具，全部改用filesystem MCP，同时也关闭了VSCode工具，问答使用interactive-mcp，关闭了完成通知工具。dida365用于TODO规划，若需要修改需要同步修改几乎所有文件。

## 适用范围

- GitHub Copilot for VS Code 
- Copilot CLI
- Visual Studio Code Agent - Insiders

## 核心能力

- 提供 `Continuous` 智能体，用于持续推进复杂开发任务、调试、重构和文档调研。
- 提供 `repo-explorer` 只读子智能体，用于快速理解仓库结构和任务落点。
- 提供 `doc-researcher` 只读子智能体，用于核实官方文档、配置和版本差异。
- 通过 `continuous-base.instructions.md` 强制使用简体中文、先读项目规则、先确认工作规划、避免盲目修改。
- 通过 Dida365 MCP 将长任务同步到滴答清单，并在项目 `.github/dida365-state.md` 中维护本地镜像。
- 通过 hook 阻止或提醒高风险行为，例如使用内置编辑/提问工具、忘记同步任务状态、未关闭 intensive chat、会话结束前缺少状态恢复信息。
- 通过 `PreCompact`、`sessionEnd`、`errorOccurred` hook 写入运行时快照，便于上下文压缩、错误恢复和会话续接。

## 目录结构

```text
continuous/
  .mcp.json                         MCP 服务配置模板
  .plugin/plugin.json               插件元信息
  agents/
    Continuous.agent.md             主智能体定义
    repo-explorer.agent.md          仓库探索子智能体
    doc-researcher.agent.md         文档调研子智能体
  hooks/
    hooks.json                      已内联脚本的 hook 配置
  rules/
    continuous-base.instructions.md 全局规则
  skills/
    context-recovery/               上下文恢复技能
    work-planning-todo-sync/        工作规划和 Dida365 同步技能
```

## 依赖

- Python 3.11+
- Node.js 24+
- `uvx`，用于运行 `mcp-server-fetch`
- 最新版本的 Copilot CLI 或 VS Code Copilot 插件
- Dida365 账号，用于远程任务状态同步
- Context7 API Key，用于官方文档检索
- 可选：Exa MCP 授权能力，用于扩展网络调研

## 快速开始：插件导入方式

1. 克隆仓库。

```shell
git clone https://github.com/Qintsg/continuous-copilot.git
cd continuous-copilot
```

2. 在 Copilot 插件管理界面中导入本地插件。

选择仓库中的 `continuous/` 目录作为插件目录。该目录包含 `.plugin/plugin.json`、`agents/`、`hooks/`、`rules/`、`skills/` 和 `.mcp.json`，可以作为完整插件包导入。

3. 配置 MCP 服务。

插件导入后，打开 Copilot 当前使用的 MCP 配置，至少完成以下配置：

- 将 `context7.env.CONTEXT7_API_KEY` 替换为你自己的 Context7 API Key。
- 确认 `dida365` 等 MCP 可登录并授权。
- 确认 `interactive-mcp` 启动参数包含 `--disable-tools message_complete_notification`。
- 按本机磁盘情况调整 `filesystem` MCP 允许访问的路径。

4. 启动 Copilot，并选择 `Continuous` 智能体。

建议在 Autopilot 模式下使用。首次进入某个项目时，智能体会先读取项目规则、README、构建配置和任务状态，再确认工作规划。

## 快速开始：导入到自己的配置

如果你不使用插件导入，也可以手动把 `continuous/` 中的内容合并到自己的 Copilot 配置目录。常见目标目录为 `~/.copilot/`。如果目标目录已有同名文件，请手动合并，不要直接覆盖已有的个人配置。

建议合并内容：

- `agents/`
- `hooks/hooks.json`
- `rules/`
- `skills/`
- `.mcp.json` 中的 `mcpServers`
- `.plugin/`

合并 MCP 配置时，只复制 `continuous/.mcp.json` 里的 `mcpServers` 条目到你自己的 MCP 配置中，并保留你已有的其他服务。合并后仍需完成以下配置：

- 将 `context7.env.CONTEXT7_API_KEY` 替换为你自己的 Context7 API Key。
- 确认 `dida365` MCP 可登录并授权。
- 确认 `interactive-mcp` 启动参数包含 `--disable-tools message_complete_notification`。
- 按本机磁盘情况调整 `filesystem` MCP 允许访问的路径。

完成后重启 Copilot 或刷新插件/配置，再选择 `Continuous` 智能体。

## Hook 说明

`continuous/hooks/hooks.json` 中的 hook 命令全部为内联 Python：

- Windows 使用 `python -X utf8 -c "..."`。
- Linux 和 macOS 使用 `python3 -c "..."`。
- 每段 Python 源码通过 `zlib + base64` 压缩嵌入，不引用 `~/.copilot/hooks/scripts/*.py`。

各 hook 的职责如下：

| Hook | 作用 |
| --- | --- |
| `SessionStart` | 判断当前目录是否像项目仓库，提示优先读取 `AGGENTS.md` / `AGENTS.md` 和 README，并解析 Dida365 项目名。 |
| `UserPromptSubmit` | 如果缺少 `.github/dida365-state.md`，提醒先确认工作规划并建立远程/本地任务状态。 |
| `PreToolUse` | 阻止内置编辑/提问工具和 `message_complete_notification`，提醒 intensive chat、Dida365 和高风险终端命令使用规则。 |
| `PostToolUse` | 文件修改、Dida365 调用、intensive chat 开关后补充状态同步提醒。 |
| `SubagentStop` | 子智能体结束前要求输出完成内容、关键发现、涉及文件和下一步建议。 |
| `Stop` | 会话结束前检查本地 Dida365 镜像、远程状态、未完成事项和 intensive chat 状态。 |
| `PreCompact` | 在上下文压缩前写入 `.github/copilot-runtime/compact-state.md`。 |
| `sessionEnd` | 会话结束时写入 `.github/copilot-runtime/session-end.md`。 |
| `errorOccurred` | 发生错误时写入 `.github/copilot-runtime/last-error.md` 并提示恢复步骤。 |

## Dida365 状态规则

每个项目会绑定一个 Dida365 项目或清单，名称必须是：

```text
mcp-<projectname>
```

`projectname` 的解析顺序：

1. 优先读取 `AGGENTS.md`。
2. 如果不存在，再读取 `AGENTS.md`。
3. 如果文件中包含 `Project Name:`、`项目名称：`、`MCP Project:` 或 `MCP项目：`，使用该值。
4. 如果没有显式名称，使用 Git 仓库名。

本地镜像文件为：

```text
.github/dida365-state.md
```

它至少应包含 MCP 项目/清单名、名称来源、远程状态是否就绪、当前阶段、当前子任务、剩余未完成项数量、intensive chat 是否仍打开、最后同步时间。

## 运行时文件

Continuous Copilot 会在目标项目中写入以下运行时文件：

```text
.github/dida365-state.md
.github/copilot-runtime/compact-state.md
.github/copilot-runtime/session-end.md
.github/copilot-runtime/last-error.md
.github/copilot-runtime/subagent-stop/*.flag
```

这些文件用于任务续接、状态恢复和子智能体收尾控制。是否提交到项目仓库，由项目自身规则决定；如果不希望提交运行时文件，可以在目标项目的 `.gitignore` 中忽略 `.github/copilot-runtime/`。

## 维护 Hooks

`continuous/hooks/hooks.json` 是自包含配置。修改 hook 行为时，需要直接更新内联命令，或在临时脚本中生成新的 `zlib + base64` 载荷后写回 `hooks.json`。

当前发布目录不包含 `hooks/scripts/`，因此安装和运行只依赖 `hooks.json` 与本机 Python。

## 验证

修改或安装后建议至少执行：

```shell
python -m json.tool continuous/hooks/hooks.json
```

然后用最小输入测试关键 hook。下面示例会读取 `hooks.json` 中的 Windows 命令并执行 `SessionStart`：

```powershell
@'
import json
import subprocess
from pathlib import Path

hooks = json.loads(Path("continuous/hooks/hooks.json").read_text(encoding="utf-8"))["hooks"]
payload = json.dumps({"cwd": "."}, ensure_ascii=False).encode("utf-8")
command = hooks["SessionStart"][0]["windows"]
result = subprocess.run(command, input=payload, shell=True, capture_output=True, timeout=15)
print(result.stdout.decode("utf-8", errors="replace"))
'@ | python -
```

在实际 Copilot 环境中，还需要验证 MCP 服务可启动、Dida365 授权可用、`filesystem` 路径允许访问你的项目目录。

## 常见问题

### `filesystem` MCP 无法修改文件

检查 `.mcp.json` 中 `filesystem.args` 允许访问的磁盘或目录。默认模板包含 `E:/`、`C:/`、`D:/`，你可以按自己的机器调整。

### Context7 不可用

确认 `CONTEXT7_API_KEY` 已替换为有效 Key，并且 Node.js 和 `npx` 可用。

## 许可

本仓库用于个人 Copilot 工作流配置。正式发布或团队共享前，请根据你的使用场景补充许可证、贡献规则和安全审查流程。

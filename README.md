# 学习 Agent 项目(learn-agent)— 从零手写一个 Claude Code

> 一个关于 Claude Code 内部机制的学习项目:20 个章节,每章只加一个机制,最终合成一个完整可运行的 coding agent。
>
> **机制很多,循环一个。**

---

## 这是什么

这是一个**学习项目**(非生产系统),目标是搞懂 Claude Code 这类 coding agent 的内部机制:工具分发、权限、hooks、todo、子 agent、技能、记忆、上下文压缩、错误恢复、任务图、后台任务、cron、团队与协议、worktree、MCP。

> **项目来源**:基于 [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)(MIT License)的第 20 章终章 `s20_comprehensive` 修改而来,详见下方[与原项目的关系](#与原项目的关系)。

一切从最小循环出发:

```python
while True:
    response = LLM(messages, tools)
    if not has_tool_use(response.content):
        return
    results = execute_tools(response.content)
    messages.append(tool_results)
```

从 s01 到 s20,代码表面越来越复杂,但核心始终没变。Claude Code 的复杂性不是"另一个 agent 大脑",而是一个成熟 harness 的复杂性:**模型负责判断和行动选择,harness 负责把环境、工具、权限、记忆、团队和外部能力组织好。**

## 与原项目的关系

本项目基于 [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code)(MIT License)的 **s20 终章**版本,在其基础上做了两类改动:

**1. Bug 修复**

- [FIX-1] 系统 prompt 与队友 prompt 中注入平台提示(Windows 环境下 shell 工具需明确使用 PowerShell)
- [FIX-2] `run_bash` / `run_git` / `_count_worktree_changes` 手动解码输出并捕获**所有**异常,工具报错不再中断主循环
- [FIX-3] `run_read` 依次尝试多种编码读取文件
- [FIX-4] 所有文件读写显式指定 `encoding="utf-8"`
- [FIX-5] 工具处理器调用统一包装异常(Lead、teammate、subagent 三处)
- [FIX-6] `MessageBus.read_inbox` 容忍 Windows 文件锁导致的 unlink 失败

**2. Windows 环境适配**

原项目面向 Unix 环境,本项目已适配 Windows(bash 工具走 PowerShell、路径/编码/文件锁问题均已处理),可在 Windows 10/11 上直接运行。

完整修复清单见 [`s20_comprehensive/code.py`](s20_comprehensive/code.py) 文件头部注释;课程章节(web 课程站、章节 README)保持上游原样未动。

## 目录结构

| 目录 | 说明 |
|------|------|
| [`s20_comprehensive/`](s20_comprehensive/) | 终章:约 2000 行 Python,把前 19 章的全部机制合成一个可运行的 agent(附中/英/日三语 README 与架构图) |
| [`web/`](web/) | Next.js 16 课程站:20 章讲解 + 交互式可视化 + Agent Loop 模拟器 + 版本 diff + 三语(zh/en/ja) |
| [`skills/`](skills/) | 4 个 Claude Code 技能:agent-builder / code-review / mcp-builder / pdf |
| [`tests/`](tests/) | pytest 测试(早期完整目录结构下运行) |

## 20 章路线图

| # | 章节 | 核心新增 |
|---|------|----------|
| s01 | Agent Loop | 最小循环:调模型 → 有 tool_use 就执行 → 结果回填 |
| s02 | Tool Use | 加一个工具只需一行:注册进分发表 |
| s03 | Permission | 危险动作执行前要有决策点 |
| s04 | Hooks | 横切逻辑挂在循环外,不写进循环里 |
| s05 | TodoWrite | 没有计划的 agent 会漂移 |
| s06 | Subagent | 一次性子 agent,隔离上下文 |
| s07 | Skill Loading | 技能目录进 prompt,按需加载 |
| s08 | Context Compact | 上下文会满,压缩是必选项 |
| s09 | Memory | 跨会话保留一层不丢细节的记忆 |
| s10 | System Prompt | 运行时组装,绝不写死 |
| s11 | Error Recovery | 429/529 重试、max_tokens 升级 |
| s12 | Task System | 大目标拆成可依赖、可认领的小任务 |
| s13 | Background Tasks | 慢操作放后台,主循环不阻塞 |
| s14 | Cron Scheduler | 定时产生工作 |
| s15 | Agent Teams | 一个 agent 不够,组队 |
| s16 | Team Protocols | 队友之间需要协议 |
| s17 | Autonomous Agents | 查任务板、认领任务 |
| s18 | Worktree Isolation | 目录隔离,互不冲突 |
| s19 | MCP Tools | 外部工具,标准协议 |
| s20 | Comprehensive | 全部机制,归到一个循环 |

## 快速开始

### 1. 课程站(推荐先看这个)

```sh
cd web
npm install
npm run dev    # 打开 http://localhost:3000
```

### 2. 运行终章 Agent

```sh
pip install anthropic python-dotenv pyyaml
# 在 .env 中配置 ANTHROPIC_API_KEY
python s20_comprehensive/code.py
```

可以试:

- `Create a todo list for inspecting this repo, then list Python files`
- `Connect to the docs MCP server and search for agent loop`
- `Create two tasks, create worktrees for them, then spawn alice and bob. Ask them to submit plans before claiming tasks.`
- `remind me of the meeting in 3 minutes.`
- `Run npm install in the background and continue reading README.md`

## 测试

```sh
pytest tests/
```

> 注:`tests/` 引用 s01–s19 的历史目录(如 `agents/`、`s05_todo_write/`),当前仓库聚焦保留终章与课程站,部分测试需还原完整章节目录后运行。

## 学习总结

1. **核心循环从未变过。** 从 s01 到 s20,代码越来越复杂,但核心始终是那个 `while True` + `tool_use` 循环。增长的复杂度属于 harness,不属于模型。
2. **模型负责判断,harness 负责组织。** 环境、工具、权限、记忆、团队、外部能力都是 harness 的职责;模型只负责决定下一步行动。
3. **权限是决策点,不是判断式。** 把 permission 做成 `PreToolUse` hook,而不是散落在工具执行行里——审计、日志都能挂到同一个点上。
4. **每章只加一个机制。** 单个机制的增量最容易看清它挂在循环的哪个位置,这也是 s20 能"全部归位"的原因。
5. **上下文会满。** 压缩管线(tool_result_budget → snip → micro → history)不是优化手段,而是长期运行 agent 的必需品。
6. **团队 = 持久队友 + 协议。** 一次性 subagent 解决"上下文隔离",持久队友解决"长期并行协作",协议解决"协作秩序"。

## 说明

- 本项目遵循上游 [shareAI-lab/learn-claude-code](https://github.com/shareAI-lab/learn-claude-code) 的 MIT 许可,原版权声明保留在 [LICENSE](LICENSE) 中。
- 代码以教学可读性优先,细节改动见上文[与原项目的关系](#与原项目的关系)。
- 技能(`skills/`)为课程配套练习:agent-builder 教如何设计 agent,code-review 教如何审查代码,mcp-builder 教如何写 MCP server,pdf 教如何处理 PDF。

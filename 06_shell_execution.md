# 第 6 章 Shell 执行

## 这个功能是什么

Shell 是 Claude Code 最强、也最危险的能力层。它让 agent 从"会改文件"升级为"会操作真实环境"——运行测试、安装依赖、启动服务、部署代码，都要经过这里。

但 Claude Code 没有把 shell 当成无边界的万能后门。它做成了有权限分级、有语义识别、有前后台策略的正式工具。

## 用户如何感知它

用户最常见的感知是：
- Claude Code 会运行 bash / powershell / REPL 命令
- 某些命令会弹出确认框，某些直接执行
- 搜索类、读取类命令的输出常被折叠，不撑开界面
- 长命令或耗时任务可以后台化，不阻塞交互

## 实现链路

一条 shell 调用通常是：

1. 模型调用 `BashTool`，传入 `command`、`description`（可选）、`timeout`（可选）、`run_in_background`（可选）
2. 工具做命令语义分析：识别是 search/read/list 这类只读命令，还是有副作用的写操作
3. 执行前做权限判断：deny rule 匹配、命令级安全规则、只读约束检查
4. 确定 sandbox 策略：是否需要容器隔离
5. 真正执行时挂到 `LocalShellTask` 任务体系，获得任务 ID 和输出文件路径
6. 结果通过 tool result message 和 background hint 呈现给用户

## 关键源码点

[`tools/BashTool/BashTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx) 说明 BashTool 的本体不是 `exec`，而是"命令语义 + 权限 + 任务 + 展示"的总封装：

- 会识别 search/read/list/silent command，用于优化 UI 折叠策略（只读命令默认 collapse）
- `run_in_background` 不是简单布尔开关，受环境变量和特性开关共同影响
- sandbox 决策在工具层做，不是全局统一配置，可以按命令类型区分处理
- 通过 `spawnShellTask`、`backgroundExistingForegroundTask` 等函数把 shell 纳入任务系统
- 结果接入 tool result storage、preview、progress message 等展示机制

**sandbox 策略**：`utils/sandbox/*` 里的逻辑决定某条命令是否需要容器隔离。不是所有命令都跑在 sandbox 里——sandbox 有性能成本，Claude Code 只在必要时启用。判断依据包括：命令类型、当前权限模式、是否有副作用。

## 为什么这样做

如果 shell 只是裸 `exec`，Claude Code 会失去四样东西：

**风险分级**：裸 exec 无法区分"搜索文件"和"删除目录"，都是同等权限。BashTool 的命令语义识别让两者有不同的确认策略。

**可解释展示**：shell 命令原始输出通常是文本流，不适合直接塞进 UI。BashTool 对输出做格式化、截断、折叠，让长输出不破坏会话体验。

**长任务管理**：长时间运行的命令需要后台化、可查看进度、可终止。裸 exec 无法提供这些，任务系统才可以。

**一致体验**：如果 shell 体验和其他工具完全不同（没有权限检查、没有 diff 展示、没有任务追踪），用户在用 Claude Code 执行 shell 命令时会觉得"进入了另一个系统"。

Claude Code 选择把 shell 正式产品化，是因为 shell 不是附属功能，而是 coding agent 的主战场之一。

## 本章关键文件
- [BashTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)
- `tools/PowerShellTool/*`
- `tools/REPLTool/*`
- `utils/sandbox/*`
- `tasks/LocalShellTask/*`

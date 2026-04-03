# 第 6 章 Shell 执行

## 这个功能是什么
Shell 是 Claude Code 最强、也最危险的能力层。它让 agent 从“会改文件”升级为“会操作真实环境”。

但 Claude Code 并没有把 shell 当成一个无边界的万能后门，而是做成了有权限、有语义识别、有前后台策略的正式工具。

## 用户如何感知它
用户最常见的感知是：
- Claude Code 会运行 bash / powershell / repl 命令
- 某些命令会要求确认
- 搜索类、读取类命令常被折叠展示
- 长命令或长任务可能被后台化

## 实现链路
一条 shell 调用通常是：
1. 模型调用 `BashTool`
2. 工具根据 schema 接收 `command`、`description`、`timeout`、`run_in_background`
3. 执行前先做命令语义解析、权限判断、只读约束和 sandbox 决策
4. 真正执行时挂到 `LocalShellTask` 任务体系
5. 结果再通过 tool result message 和 background hint 呈现给用户

## 关键源码点
[`tools/BashTool/BashTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx) 很能说明 Claude Code 的产品判断：
- 会识别 search/read/list/silent command，用来优化 UI 表现
- `run_in_background` 不是简单布尔开关，而受环境变量和特性开关影响
- 会在工具层判断 sandbox、foreground/background、task output 存储
- 通过 `spawnShellTask`、`backgroundExistingForegroundTask` 等任务函数把 shell 纳入任务系统
- 结果会接入 tool result storage、preview、progress message 等展示机制

所以 BashTool 的本体并不是 `exec`，而是“命令语义 + 权限 + 任务 + 展示”的总封装。

## 为什么这样做
如果 shell 只是裸 `exec`，Claude Code 会失去：
- 风险分级
- 可解释展示
- 长任务管理
- 与其他功能的一致体验

Claude Code 选择把 shell 正式产品化，是因为它知道 shell 不是附属功能，而是 coding agent 的主战场之一。

## 本章关键文件
- [BashTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)
- `tools/PowerShellTool/*`
- `tools/REPLTool/*`
- `utils/sandbox/*`
- `tasks/LocalShellTask/*`

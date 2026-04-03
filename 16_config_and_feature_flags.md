# 第 16 章 Config 与 Feature Flags

## 这个功能是什么
Claude Code 的配置系统不仅控制参数，还控制“哪些功能暴露给谁”。feature flag 则是它承载多产品线、多实验能力的骨架。

## 用户如何感知它
用户经常会间接感知到：
- 不同环境、不同组织、不同构建的功能不一样
- 某些命令只在内部或实验版出现
- 某些运行模式是被整体裁掉的

## 实现链路
功能暴露控制通常发生在两层：
1. 入口层：某些运行模式根本不被暴露
2. 命令层和工具层：某些命令、参数或能力被条件加载或条件裁剪

## 关键源码点
[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 里很多 fast path 都包在 `feature(...)` 里，例如 `DAEMON`、`BRIDGE_MODE`、`CHICAGO_MCP`。

[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 更明显：
- 大量命令通过 `feature(...) ? require(...) : null` 注册
- `INTERNAL_ONLY_COMMANDS` 显式维护内部命令集合
- 某些懒加载命令明显是为减小主路径成本而设计的

再看 [`tools/AgentTool/AgentTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx) 和 [`tools/BashTool/BashTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)，连输入 schema 都会受 feature flag 和环境变量影响。

## 为什么这样做
Claude Code 不是单一产品，它更像一条产品族：
- 外部版
- 内部版
- 实验版
- 组织定制版

没有 feature flag，这种代码库很难维持。
有了 feature flag，系统才可能在一份主干上并行演化多条能力路线。

## 本章关键文件
- [cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)
- [AgentTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx)
- [BashTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)

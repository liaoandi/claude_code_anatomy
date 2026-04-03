# 第 16 章 Config 与 Feature Flags

## 这个功能是什么

Claude Code 的配置系统不只控制参数，还控制"哪些功能暴露给谁"。Feature flag 是它承载多产品线、多实验能力的骨架——同一份代码，不同开关，不同的产品形态。

## 用户如何感知它

用户通常间接感知到：
- 不同环境、不同组织、不同构建的功能集不一样
- 某些命令只在内部版或实验版出现
- 某些运行模式在特定构建里被整体裁掉

极少数情况下用户会直接感知：通过 `/config` 查看或修改运行时配置。

## 实现链路

功能暴露控制通常发生在两层：

**入口层**：某些运行模式根本不被暴露，`feature(...)` 包裹整个分支，feature flag 为 false 时编译器可以裁掉整块代码。

**命令层和工具层**：某些命令、参数或能力通过条件注册实现，feature flag 控制注册而不是运行时判断。

## 关键源码点

[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 里很多 fast path 都包在 `feature(...)` 里，例如 `DAEMON`、`BRIDGE_MODE`、`CHICAGO_MCP`。这些不只是运行时判断，还是 dead code elimination 的依据。

[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 更明显：
- 大量命令通过 `feature(...) ? require(...) : null` 注册——feature flag 为 false 时，模块根本不加载
- `INTERNAL_ONLY_COMMANDS` 维护内部命令集合，不暴露给外部用户
- 懒加载命令明显为减小主路径加载成本而设计

再看 [`tools/AgentTool/AgentTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx) 和 [`tools/BashTool/BashTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)：连输入 schema 的字段也受 feature flag 和环境变量影响，说明 feature flag 不只在命令层，还渗透到工具行为层。

## 为什么这样做

Claude Code 不是单一产品，更像一条产品族：
- 外部公开版
- 内部研发版
- 实验功能版
- 组织定制版

没有 feature flag，这条产品族的代码库很难维持。有了 feature flag，系统才可能在一份主干上并行演化多条能力路线，而不是维护多个 fork。

feature flag 结合 dead code elimination 还有另一个好处：不同构建版本的包体积可以差异很大，外部版不携带任何内部功能的代码。

## 本章关键文件
- [cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)
- [AgentTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx)
- [BashTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/BashTool/BashTool.tsx)

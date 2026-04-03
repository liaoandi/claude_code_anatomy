# 第 3 章 Slash Commands 功能族

## 这个功能是什么

Slash command 是 Claude Code 把高频能力产品化的显式入口。它不是 shell cosplay，而是"让高确定性功能拥有稳定接口"。

这一章只讲命令系统本身，不重复展开第 1 章的 CLI 启动分流。

## 用户如何感知它

熟练用户使用 slash command 的典型场景有三类：
- **切模式**：`/plan` 进入规划模式，`/compact` 压缩上下文
- **打开功能**：`/memory` 编辑记忆文件，`/mcp` 管理 MCP server
- **查询状态**：`/permissions` 查看权限规则，`/config` 查看配置

这些操作的共同点是：用户不想把它们留给模型猜。`/plan` 就是 `/plan`，不是"请帮我进入规划模式"。

## 实现链路

一条命令链路大致是：

1. [`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 注册全部可见命令
2. 命令按类型分成 prompt command（注入 prompt 后交给模型）、local JSX command（直接返回 UI）等不同形式
3. 某些命令静态引入，某些按 feature flag 懒加载（`feature(...) ? require(...) : null`）
4. 命令执行时进入各自目录下的 `call` 或 prompt 构造逻辑
5. 插件命令、skill 命令也会并入命令层，共享同一套注册机制

## 关键源码点

[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 暴露出几件重要的事：

- **命令量很大，已接近产品能力目录**：`/plan`、`/memory`、`/compact`、`/context`、`/permissions`、`/mcp`、`/review`……每个都是独立功能的入口
- **`INTERNAL_ONLY_COMMANDS`** 明确把内部命令与外部能力分开，用户看不到内部命令
- **feature-gated 命令**用 `require(...) : null` 实现 dead code elimination 友好加载，功能开关和代码加载同步
- **插件命令、skill 命令、builtin plugin skill commands** 也并入命令层，说明命令系统是 Claude Code 对"功能表面"做统一治理的地方

## 为什么这样做

Claude Code 保留 slash command，本质上是在承认一件事：高阶 AI 产品不应该只优化自由表达，还要优化熟练用户的高频确定性操作。

slash command 带来三种产品价值：
- **可发现**：`/` 触发后的候选列表就是能力目录
- **可预测**：相同命令每次行为一致，不受模型随机性影响
- **可形成肌肉记忆**：熟练后不需要思考，直接打命令

去掉 slash command，你得到的不是更纯粹的 AI 体验，而是把能力推回到了自然语言的不确定性里。

## 本章关键文件
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)
- [commands/plan/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/index.ts)
- [commands/memory/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/index.ts)
- [commands/permissions/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/index.ts)

# 第 1 章 对话与命令输入

## 这个功能是什么

Claude Code 的第一层能力不是生成回答，而是把用户输入送到正确的执行路径。

表面上只有两种入口：
- 自然语言输入
- slash command

但从源码看，这两者只在产品表面上并列，在运行时里属于不同层次。自然语言是任务意图，slash command 是显式调度。两者职责不同，不互相替代。

## 用户如何感知它

重度用户会形成两套操作心智：
- **目标表达**：直接输入一句话，让模型决定怎么做
- **功能调用**：直接打 `/plan`、`/memory`、`/config`，触发确定性功能

这说明 Claude Code 不想把所有能力都藏在 prompt 猜测里。它故意保留了一层高确定性的功能入口，让熟练用户不依赖自然语言推断就能直接触达系统能力。

## 实现链路

一条最典型的输入链路是：

1. CLI 进程进入 [`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
2. 入口先判断是否 fast path，例如 `--version`（零模块加载直接退出）
3. 再判断是否特殊运行模式：`daemon`、`bridge`、`ps/logs/attach/kill`
4. 只有普通交互路径才继续加载完整主程序
5. 进入交互会话后，slash command 由 [`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 的命令注册表承接

这个顺序很重要：Claude Code 先分"运行模式"，再分"交互内容"。运行模式的分流发生在比命令注册早得多的位置。

## 关键源码点

[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 有三个值得注意的设计：
- `--version` 是零额外模块加载的 fast path，说明团队在认真控制冷启动时间
- `bridge`、`daemon`、`bg sessions` 在入口层直接分流，不进入主应用初始化
- 大量逻辑用动态 `import()`，只在实际需要时才加载

[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 则说明：
- 命令是结构化对象，不是散乱函数
- 命令注册中心统一控制"哪些命令存在、哪些可见"
- feature flag 深度参与命令是否暴露给用户
- 某些大模块被 lazy shim 包起来，只在实际调用时加载

## 为什么这样做

如果 Claude Code 只有自然语言入口，熟练用户会失去确定性——每次都要猜模型理不理解你的意图。

如果只有命令入口，它又退回传统 CLI，失去 agent 的灵活性。

Claude Code 采用"双入口、双职责"设计：
- 自然语言负责表达目标
- slash command 负责触发确定性功能

这不是折中，而是把 agent 体验和专家工作流同时保留下来的刻意选择。

## 本章关键文件
- [entrypoints/cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)

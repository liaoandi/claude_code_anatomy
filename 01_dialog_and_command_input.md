# 第 1 章 对话与命令输入

## 这个功能是什么
Claude Code 的第一层能力不是生成回答，而是把用户输入送到正确的执行路径。

表面上你只看到两种入口：
- 自然语言输入
- slash command

但从源码看，这两者只是在产品表面上并列，在运行时里并不属于同一层。自然语言更像任务意图，slash command 更像显式调度。

## 用户如何感知它
重度用户会形成两套心智：
- 目标表达：直接输入一句话
- 功能调用：直接打 `/plan`、`/memory`、`/config`、`/review`

这说明 Claude Code 不想把所有能力都藏在 prompt 猜测里，而是故意保留了一层高确定性的功能入口。

## 实现链路
一条最典型的输入链路是：
1. CLI 进程先进入 [`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
2. 入口先判断是不是 fast path，例如 `--version`
3. 再判断是不是特殊运行模式，例如 `daemon`、`bridge`、`ps/logs/attach/kill`
4. 只有普通交互路径才继续加载完整 CLI 主程序
5. 进入交互会话后，slash command 再由 [`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 的命令注册表承接

这个顺序很重要。Claude Code 先分“运行模式”，再分“交互内容”。

## 关键源码点
[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 有三个很值得注意的设计：
- `--version` 是零额外模块加载的 fast path
- `bridge`、`daemon`、`bg sessions` 都在入口层直接分流
- 大量逻辑用动态 `import()`，明显在压冷启动成本

[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 则说明：
- 命令是结构化对象，不是散乱函数
- 命令注册中心统一控制“哪些命令存在”
- feature flag 深度参与命令是否暴露
- 某些大模块被 lazy shim 包起来，只在实际调用时加载

## 为什么这样做
如果 Claude Code 只有自然语言入口，熟练用户会失去确定性。
如果 Claude Code 只有命令入口，它又会退回传统 CLI。

所以 Claude Code 采用的是“双入口、双职责”设计：
- 自然语言负责表达目标
- slash command 负责触发确定性功能

这不是中庸，而是把 agent 体验和专家工作流同时保留下来。

## 本章关键文件
- [entrypoints/cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)

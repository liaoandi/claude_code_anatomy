# 第 3 章 Slash Commands 功能族

## 这个功能是什么
Slash command 是 Claude Code 把高频能力产品化的显式入口。它不是 shell cosplay，而是“让高确定性功能拥有稳定接口”。

这一章只讲命令系统本身，不重复展开 CLI 启动分流。

## 用户如何感知它
熟练用户使用 slash command 的典型场景有三类：
- 切模式，例如 `/plan`
- 打开某项功能，例如 `/memory`
- 查询或调整当前状态，例如 `/permissions`、`/config`

这些操作的共同点是：用户不想把它们留给模型猜。

## 实现链路
一条命令链路大致是：
1. [`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 注册全部可见命令
2. 命令按类型分成 prompt command、local JSX command 等不同风格
3. 某些命令直接静态引入，某些命令按 feature flag 延迟 `require`
4. 命令实际执行时，再进入各自目录下的 `call` 或 prompt 构造逻辑

这意味着 slash command 不是“字符串匹配后执行函数”，而是完整的本地功能协议。

## 关键源码点
[`commands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts) 暴露出几件重要的事：
- 命令注册量很大，已经接近“产品能力目录”
- `INTERNAL_ONLY_COMMANDS` 明确把内部命令与外部能力分开
- 大量 feature-gated 命令用 `require(...) : null` 的方式做 dead code elimination 友好加载
- 插件命令、skill 命令、builtin plugin skill commands 也会并入命令层

也就是说，slash command 不只是手工写死的一组命令，而是 Claude Code 对“功能表面”做统一治理的地方。

## 为什么这样做
Claude Code 保留 slash command，本质上是在承认一件事：
高阶 AI 产品不应该只优化自由表达，还要优化熟练用户的高频确定性操作。

slash command 带来的不是“更传统”，而是三种产品价值：
- 可发现
- 可预测
- 可形成肌肉记忆

## 本章关键文件
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)
- [commands/plan/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/index.ts)
- [commands/memory/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/index.ts)
- [commands/permissions/index.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/index.ts)

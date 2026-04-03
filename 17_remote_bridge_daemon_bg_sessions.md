# 第 17 章 Remote、Bridge、Daemon、Bg Sessions

## 这个功能是什么
Claude Code 并不把自己限制成“前台 terminal 里的一次性 REPL”。它还支持后台会话、桥接环境、远程控制和 supervisor 模式。

这些能力合在一起，说明 Claude Code 想做的是长期运行的工作环境，而不是单次调用工具。

## 用户如何感知它
用户会在以下场景里感知这些模式：
- 用 `ps/logs/attach/kill` 管理后台 session
- 让任务挂后台继续跑
- 通过 bridge/remote-control 连接别的环境
- 用 daemon 维持一个长期 supervisor

## 实现链路
这些模式的共同特点是：
1. 都在入口层直接分流
2. 都在加载主应用前决定自己是不是独立运行模式
3. 都不愿意和普通 REPL 共享同一条初始化路径

## 关键源码点
[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 很清楚地把这些模式视为一级分支：
- `remote-control` / `bridge`
- `daemon`
- `ps` / `logs` / `attach` / `kill`
- `--bg` / `--background`

而且这些分支通常都先做：
- `enableConfigs()`
- 某些模式的 auth / policy / min-version 检查
- 再单独进入各自的 `bridgeMain`、`daemonMain` 或 `bg` handlers

这说明 Claude Code 的产品判断是：
这些不是“附加命令”，而是完全不同的运行模型。

## 为什么这样做
如果把后台会话、bridge、daemon 都当成普通命令挂到 REPL 里，系统会马上出现状态污染：
- 初始化要求不同
- 权限要求不同
- 输出模型不同
- 生命周期也不同

入口层分流是更干净的做法。它让 Claude Code 能同时拥有交互式前台模式和长期运行模式，而不把它们搅在一起。

## 本章关键文件
- [cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- `bridge/*`
- `remote/*`
- `daemon/*`
- `cli/bg.js`

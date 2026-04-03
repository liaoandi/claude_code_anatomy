# 第 17 章 Remote、Bridge、Daemon、Bg Sessions

## 这个功能是什么

Claude Code 不把自己限制成"前台 terminal 里的一次性 REPL"。它还支持后台会话、桥接环境、远程控制和 supervisor 模式。

这些能力合在一起，说明 Claude Code 的目标是长期运行的工作环境，而不是单次调用工具。

## 用户如何感知它

用户会在以下场景里感知这些模式：
- 用 `ps/logs/attach/kill` 管理后台 session
- 让任务挂后台继续跑，关掉终端也不丢失
- 通过 bridge/remote-control 在另一个环境里控制 Claude Code
- 用 daemon 维持一个长期 supervisor，管理多个会话

## 实现链路

这些模式的共同特点是：

1. 都在入口层直接分流，不进入普通 REPL 初始化
2. 都在加载主应用前决定自己的运行模式
3. 各有自己的初始化序列（auth、policy、min-version 检查等）

## 关键源码点

[`entrypoints/cli.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx) 把这些模式视为一级分支：

```
remote-control / bridge  → bridgeMain()
daemon                   → daemonMain()
ps / logs / attach / kill → bg session management
--bg / --background       → background session handlers
```

每个分支通常先做：
- `enableConfigs()`：加载配置
- auth / policy / min-version 检查：确保环境满足要求
- 再单独进入各自的 main handler

这说明 Claude Code 的产品判断是：这些不是"附加命令"，而是完全不同的运行模型，有独立的初始化要求。

**各模式的用途**：
- **bridge**：让另一个进程（如 IDE 插件）通过标准协议控制 Claude Code，实现双向通信
- **daemon**：长期运行的 supervisor 进程，管理多个 Claude Code 会话的生命周期
- **bg sessions**：后台会话，独立于终端存在，`attach` 后可以继续交互
- **remote-control**：从远端发送指令给本地 Claude Code 实例，适合 CI/CD 场景

## 为什么这样做

如果把后台会话、bridge、daemon 当成普通命令挂到 REPL 里，会出现状态污染：
- 初始化要求不同（daemon 需要比 REPL 更早的 config 加载）
- 权限要求不同（bridge 模式有 bridgePermissionCallbacks）
- 输出模型不同（后台会话输出到文件，不是 terminal）
- 生命周期也不同（daemon 持久运行，REPL 随用随退）

入口层分流是更干净的做法：不同运行模型的代码路径完全独立，不会相互污染，也更容易单独测试和维护。

## 本章关键文件
- [cli.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/entrypoints/cli.tsx)
- `bridge/*`
- `remote/*`
- `daemon/*`
- `cli/bg.js`

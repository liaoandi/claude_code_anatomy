# 第 17 章 Remote、Bridge、Daemon、Bg Sessions

## Claude Code 不只是一个 REPL

大多数时候你用 Claude Code 的方式是：打开终端，开始对话，关掉终端，结束。

但 Claude Code 还支持几种不同的运行模式，适合更复杂的使用场景：

**后台会话（bg sessions）**：任务放到后台跑，关掉终端也不会停。之后可以用 `attach` 重新连上去，用 `logs` 查看输出，用 `kill` 终止。适合长时间运行的任务。

**Bridge**：让另一个程序——比如 IDE 插件——通过标准协议控制 Claude Code。VS Code 插件就是这样和 Claude Code 通信的。Bridge 模式有自己的权限回调机制，允许 IDE 拦截并处理权限确认。

**Daemon**：一个长期运行的 supervisor 进程，管理多个 Claude Code 会话的生命周期。适合需要在同一台机器上跑多个会话、需要持久状态的场景。

**Remote control**：从远端发送指令给本地的 Claude Code 实例。适合 CI/CD 流程或者自动化脚本。

## 为什么这些模式在入口层分流

这些运行模式有一个共同点：它们的初始化要求和普通交互 REPL 不同。

Daemon 需要比 REPL 更早加载某些配置。Bridge 需要注册特殊的权限回调。后台会话需要把输出重定向到文件，不往 terminal 写。

如果把这些都当成普通命令处理，初始化过程里就会充满"如果是 bridge 模式就做这个，如果是 daemon 就做那个"的判断，代码会很乱。

入口层分流让每种模式都走自己的初始化路径，互不干扰。普通 REPL 不会加载 bridge 相关的代码，bridge 模式不会走 REPL 的初始化流程。

## 这些模式和 task 体系的关系

后台会话、remote agent 这些都和第 9 章讲的 task 体系相关。`remote_agent` 类型的 task 就是在远程环境里运行的 agent，通过 bridge 或 remote 机制通信。

从用户角度看，管理一个后台 session 和管理一个后台 task 体验类似：都可以 attach、kill、查看 logs。这是因为底层用的是同一套机制。

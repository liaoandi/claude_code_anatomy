# 第 17 章 Remote、Bridge、Daemon、Bg Sessions

## Claude Code 不只是一个对话窗口

大多数时候你用 Claude Code 的方式是：打开终端，开始对话，关掉终端，结束。

但 Claude Code 还支持几种不同的运行模式：

- **Bg Sessions（后台会话）**：任务放到后台运行，关掉终端也不会停。可以随时用 `attach` 重新连上查看进度，用 `logs` 看历史输出，用 `kill` 终止。适合耗时较长的任务。
- **Bridge**：让另一个程序通过标准协议控制 Claude Code。VS Code 插件就是走这条路和 Claude Code 通信的。Bridge 模式有独立的权限回调机制，IDE 可以拦截权限确认并自行处理。
- **Daemon**：一个持续运行的后台进程，负责管理多个 Claude Code 会话的生命周期。适合需要同时跑多个会话、或者需要跨会话保持状态的场景。
- **Remote Control**：从远端向本地的 Claude Code 实例发送指令，适合嵌入 CI/CD 流程或自动化脚本。

## 为什么这些模式在入口层分流

这些运行模式有一个共同点：它们的初始化要求和普通交互模式不同。

Daemon 需要比普通模式更早加载某些配置。Bridge 需要注册特殊的权限回调。后台会话需要把输出重定向到文件，不往 terminal 写。

如果把这些都当成普通命令处理，初始化过程里就会充满"如果是 bridge 模式就做这个，如果是 daemon 就做那个"的判断，代码会很乱。

入口层分流让每种模式都走自己的初始化路径，互不干扰。普通对话模式不会加载 bridge 相关的代码，bridge 模式不会走普通对话的初始化流程。

## 这些模式和 task 体系的关系

后台会话、remote agent 这些都和第 9 章讲的 task 体系相关。`remote_agent` 类型的 task 就是在远程环境里运行的 agent，通过 bridge 或 remote 机制通信。

从用户角度看，管理一个后台 session 和管理一个后台 task 体验类似：都可以 attach、kill、查看 logs。这是因为底层用的是同一套机制。

这里面没有传统的 Unix daemon——bridge loop 本身就是持久进程，通过一个指针文件让其他进程发现活跃会话，绕开了经典 daemon 模式的复杂性。

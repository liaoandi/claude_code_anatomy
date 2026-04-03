# 第 9 章 Task、Agent、Teammate

## 这个功能是什么

Claude Code 不只是单线程对话工具。它把很多工作包装成 task，再在 task 之上承载 agent、teammate、remote agent 等能力。

task 是承载"长生命周期执行"的基础抽象。没有它，shell、subagent、remote job、teammate 都只能各自维护一套生命周期，系统很快失控。

## 用户如何感知它

用户会在这些场景里感知到 task：
- 后台运行的 shell 命令或 agent，可以查看进度和日志
- 子 agent 或 teammate 产出独立输出，可以追踪和附着
- 可终止（kill）、可附着（attach）、可查看历史（logs）的异步任务

## 实现链路

一条 task 化链路通常是：

1. 某个工具或命令决定要启动长任务（shell 命令超过一定时长、agent 调用、remote job）
2. 系统分配 task id、output file、初始状态
3. 任务进入 `pending → running → completed/failed/killed` 生命周期
4. 前台 UI 只消费 task 状态和输出，不直接耦合底层执行者

不同 TaskType 的生命周期有所差异：
- `local_bash`：执行本地 shell 命令，完成即终止
- `local_agent`：在本地运行 subagent，有独立上下文
- `remote_agent`：在远程环境执行，通过 bridge 通信
- `in_process_teammate`：进程内并行 agent，共享部分状态

## 关键源码点

[`Task.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/Task.ts) 给出 Claude Code 对 task 的最小定义：

- 有明确的 `TaskType`（`local_bash`、`local_agent`、`remote_agent`、`in_process_teammate`）
- 有统一 `TaskStatus`（`pending`、`running`、`completed`、`failed`、`killed`）
- 每个 task 都有 `outputFile`、`outputOffset`、`notified` 等字段——输出持久化到磁盘，不在内存里
- `generateTaskId` 和类型前缀说明 task id 是对外可追踪的唯一标识

[`tools/AgentTool/AgentTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx) 说明 agent 能力是怎么落在 task 体系上的：

- AgentTool 支持 sync（等待结果）/ async（立即返回 task id）两类返回模式
- 可以 worktree 隔离（给 agent 一个独立的 git worktree），也可以在 remote 环境启动
- 背景运行、进度更新、结果输出都统一落到 task 生命周期里，不需要 AgentTool 自己维护状态

## 为什么这样做

把 task 做成一等对象，直接带来三种收益：

**长任务可管理**：有 task id 才能 attach、kill、logs。没有统一抽象，后台命令跑起来就失控了。

**多执行者统一呈现**：shell task、agent task、remote task 在 UI 里用同一套组件显示进度和状态，用户不需要区分底层是什么在跑。

**前台和后台解耦**：前台 UI 只读 task 状态和 outputFile，不关心底层是 bash 还是 agent 还是 remote job。这让执行者可以替换，不影响 UI。

## 本章关键文件
- [Task.ts](/Users/antonio/Desktop/cc2.1.88/all/src/Task.ts)
- [AgentTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx)
- `tasks/*`
- `commands/agents/*`

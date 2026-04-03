# 第 9 章 Task、Agent、Teammate

## Task 是什么

当 Claude Code 需要执行一个超过几秒的操作——跑一个 shell 命令、启动一个子 agent、连接一个远程任务——它不会直接执行然后等结果，而是创建一个 task。

Task 是 Claude Code 对"长时间执行"的统一抽象。每个 task 有唯一 id、类型、状态，输出写到磁盘上的专用文件里。状态从 `pending` 到 `running`，最终到 `completed`、`failed` 或 `killed`。

这个抽象的存在让前台 UI 和后台执行解耦：UI 只读 task 状态和输出文件，不关心底层是 bash 还是 agent 还是远程进程在跑。

## 几种 task 类型

`local_bash` 是本地 shell 命令，最简单，跑完就结束。

`local_agent` 是在本地启动的子 agent。子 agent 有自己的上下文，可以独立读文件、改文件、跑命令，完成后把结果汇报给主 agent。这是 Claude Code 实现并行工作的基础——多个子 agent 同时处理不同部分，主 agent 汇总结果。

`remote_agent` 是在远程环境启动的 agent，通过 bridge 通信。适合在 CI/CD 环境或者和主机隔离的容器里执行任务。

`in_process_teammate` 是进程内的并行 agent，比 `local_agent` 开销更小，但隔离程度也更低。

## Agent 能干什么

AgentTool 是 Claude Code 的"让另一个 AI 来做这件事"的工具。

调用时可以选择同步（等 agent 完成再继续）或者异步（提交任务，拿到 task id，继续干别的）。可以给 agent 分配一个独立的 git worktree，让它在自己的分支上工作，不影响主工作区。

子 agent 的能力和主 agent 是一样的，可以读写文件、跑命令，有自己的权限上下文。但它是一个独立的执行单元，不共享主 agent 的对话历史，只接受任务描述和必要的上下文。

## 为什么要有这一层抽象

没有 task 抽象的话，shell 命令、子 agent、远程任务都得各自管理自己的生命周期，各自实现进度显示、错误处理、输出收集。代码会很乱，用户体验也会碎片化。

有了统一的 task 层：
- 所有长时间操作都可以 attach、kill、查看 logs
- UI 只需要一套组件来显示任务状态，不管底层是什么
- 执行者可以替换（比如从 local_agent 换成 remote_agent），UI 不用改

多了一层抽象，多了复杂度，但换来的是整个系统的可管理性。

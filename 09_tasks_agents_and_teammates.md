# 第 9 章 Task、Agent、Teammate

## 这个功能是什么
Claude Code 不只是单线程对话工具，它把很多工作包装成 task，再在 task 之上承载 agent、teammate、remote agent 等能力。

也就是说，task 是它承载“长生命周期执行”的基础抽象。

## 用户如何感知它
用户会在这些场景里感知到 task：
- 后台运行的 shell 或 agent
- 子 agent 或 teammate 的独立输出
- 可附着、可终止、可通知的异步任务

## 实现链路
一条 task 化链路通常是：
1. 某个工具或命令决定要启动长任务
2. 系统分配 task id、output file、状态字段
3. 任务进入 `pending -> running -> completed/failed/killed` 生命周期
4. 前台 UI 只看 task 状态和输出，不直接耦合底层执行者

## 关键源码点
[`Task.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/Task.ts) 给出了 Claude Code 对 task 的最小定义：
- 有明确的 `TaskType`，例如 `local_bash`、`local_agent`、`remote_agent`、`in_process_teammate`
- 有统一 `TaskStatus`
- 每个 task 都有 `outputFile`、`outputOffset`、`notified` 等字段
- `generateTaskId` 和类型前缀说明 task id 是对外可追踪对象

而 [`tools/AgentTool/AgentTool.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx) 又说明 agent 能力是怎么落在 task 体系上的：
- AgentTool 支持 sync / async 两类返回
- 可以 worktree 隔离，也可以 remote 启动
- 会把 agent 注册进本地或远程任务体系
- 背景运行、进度更新、结果输出都统一落到 task 生命周期里

## 为什么这样做
如果 Claude Code 没有 task 抽象，shell、subagent、remote job、teammate 都只能各自维护一套生命周期，系统很快会失控。

把 task 做成一等对象，直接带来三种收益：
- 长任务可管理
- 多执行者可统一呈现
- 前台 UI 和后台执行解耦

## 本章关键文件
- [Task.ts](/Users/antonio/Desktop/cc2.1.88/all/src/Task.ts)
- [AgentTool.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/tools/AgentTool/AgentTool.tsx)
- `tasks/*`
- `commands/agents/*`

# 第 7 章 Plan Mode

## 这个功能是什么

Plan Mode 是 Claude Code 给复杂任务加的一层显式控制模式。它不是"先解释一下再做"，而是把系统切到一种更受约束的任务推进状态——在这个状态里，模型只规划，不执行。

## 用户如何感知它

用户会感知到几件事：
- `/plan` 会进入一个单独模式，此后模型不再直接调用写操作工具
- 当前 plan 可以查看，也可以用 `/plan open` 在外部编辑器里直接修改
- 明确退出 plan mode 后，才进入执行阶段

这种"先规划后执行"的边界不是约定，而是运行时状态，系统强制执行。

## 实现链路

Plan Mode 的链路并不复杂，但很有代表性：

1. 用户触发 `/plan`
2. [`commands/plan/plan.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx) 检查当前 `toolPermissionContext.mode`
3. 如果不在 plan mode，调用 `prepareContextForPlanMode` 和 `applyPermissionUpdate` 切换权限上下文
4. 如果已在 plan mode，读取当前 plan 文件并展示内容
5. `/plan open` 调用外部编辑器直接打开 plan 文件

进入 plan mode 后，权限上下文变化让写操作工具（FileEditTool、BashTool 有副作用的命令等）无法执行，不需要模型"自我约束"。

## 关键源码点

[`commands/plan/plan.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx) 说明 Plan Mode 的本质是权限上下文切换，不是纯 UI 状态：

- 状态写进 `toolPermissionContext`，而不是 React state
- `handlePlanModeTransition` 和 `prepareContextForPlanMode` 会影响后续所有工具的执行策略
- plan 内容来自专门的 plan 文件路径，持久化到磁盘，不是临时内存变量
- `renderToString(display)` 说明 plan 也被当作本地命令结果来呈现，进入消息流

`tools/EnterPlanModeTool` 和 `tools/ExitPlanModeTool` 让模型自己也能触发模式切换，不只是用户操作才能控制。

## 为什么这样做

复杂任务最怕两件事：
- 模型边想边做，直接越过关键决策点，出了问题难以回溯
- 用户失去"现在是在规划还是在执行"的边界感，不知道下一步会发生什么

Claude Code 把 plan mode 做成显式模式而不是口头约定，目的是让"先规划后执行"真正变成运行时约束，而不是 prompt 文案。

这个设计的副产品是：plan 文件可以在外部编辑器里手工修改，用户可以直接介入规划过程，不是只能接受模型给出的方案。

## 本章关键文件
- [plan.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx)
- `tools/EnterPlanModeTool/*`
- `tools/ExitPlanModeTool/*`

# 第 7 章 Plan Mode

## 这个功能是什么
Plan Mode 是 Claude Code 给复杂任务加的一层显式控制模式。它不是“先解释一下再做”，而是把系统切到一种更受约束的任务推进状态。

## 用户如何感知它
用户会感知到几件事：
- `/plan` 会进入一个单独模式
- 当前 plan 可以查看，也可以 `open` 到外部编辑器里改
- 有些任务在进入执行前，需要先明确计划

## 实现链路
Plan Mode 的链路并不复杂，但很有代表性：
1. 用户触发 `/plan`
2. [`commands/plan/plan.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx) 检查当前 `toolPermissionContext.mode`
3. 如果不在 plan mode，就调用 `prepareContextForPlanMode` 和 `applyPermissionUpdate` 切换权限上下文
4. 如果已经在 plan mode，就读取当前 plan 文件并展示
5. `/plan open` 则调用外部编辑器直接打开 plan 文件

## 关键源码点
[`commands/plan/plan.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx) 说明 Plan Mode 的本质是“权限上下文切换”：
- 它不是一个纯 UI 状态，而是写进 `toolPermissionContext`
- `handlePlanModeTransition` 和 `prepareContextForPlanMode` 说明它会影响后续执行策略
- plan 内容来自专门的 plan 文件路径，而不是临时内存变量
- `renderToString(display)` 说明 plan 也被当作本地命令结果来呈现

## 为什么这样做
复杂任务最怕两件事：
- 模型边想边做，直接越过关键决策点
- 用户失去“现在是在计划，还是在执行”的边界感

Claude Code 把 plan mode 做成显式模式，而不是一句口头约定，目的就是让“先规划后执行”真正变成运行时状态，而不是 prompt 文案。

## 本章关键文件
- [plan.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/plan/plan.tsx)
- `tools/EnterPlanModeTool/*`
- `tools/ExitPlanModeTool/*`

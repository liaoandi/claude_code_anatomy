# 第 12 章 Context 管理

## 这个功能是什么
Context 管理解决的是“当前窗口里什么该留、什么该压缩、用户怎么看到这件事”。

这一章讲当前上下文预算控制，不讲历史续接和长期 memory 沉淀。

## 用户如何感知它
用户会在这些地方直接看到 context 管理：
- `/compact`
- `/context`
- context visualization
- 某些对话中自动发生的 compact 或 collapse

## 实现链路
上下文管理通常有三层：
1. 命令层：允许用户主动查询和触发 compact
2. 服务层：自动 compact / collapse / token budget 计算
3. 可视化层：把“上下文花在哪里”呈现给用户

## 关键源码点
[`components/ContextVisualization.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/ContextVisualization.tsx) 很值得看，因为它把一个通常藏在系统内部的东西做成了显式产品界面：
- 会按 category 展示 context usage
- 会区分 model、memory files、mcp tools、skills、agents、message breakdown
- 如果启用了 context collapse，还会通过 `CollapseStatus` 告诉用户有多少内容被 summarized / staged

这说明 Claude Code 并不想让 context budget 变成黑箱。它试图把“上下文去哪了”讲清楚。

## 为什么这样做
长任务里最难受的体验之一，就是用户不知道系统为什么突然忘了什么、压了什么、保留了什么。

Claude Code 把 context 管理做成显式功能，有三个好处：
- 降低黑箱感
- 让压缩行为可解释
- 让高级用户能主动管理预算，而不是完全被动接受

## 本章关键文件
- [ContextVisualization.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/ContextVisualization.tsx)
- `commands/compact/*`
- `commands/context/*`
- `services/compact/*`

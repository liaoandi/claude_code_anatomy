# 第 11 章 Memory 功能

## 这个功能是什么
Claude Code 的 memory 不是简单“存一下”，而是把当前会话里真正值得保留的信息沉淀成更长期的材料。

这一章讲长期记忆与后台提取，不讲 history 续接和 compact。

## 用户如何感知它
用户会感知到三种层次：
- `/memory` 打开或编辑 memory 文件
- 某些对话会自动沉淀到 session memory
- 某些内容可能进一步进入 team memory 或更长期同步流程

## 实现链路
memory 的链路分成前台和后台两段：
1. 前台命令入口：用户通过 `/memory` 打开 memory 文件
2. 后台提取链路：系统在合适时机自动抽取会话记忆并写回 markdown 文件

## 关键源码点
[`commands/memory/memory.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/memory.tsx) 是前台入口：
- 先 `clearMemoryFileCaches()` 再 `getMemoryFiles()`，避免初次打开闪烁
- 打开时会确保目录和文件存在
- 最后把 memory 文件交给外部编辑器处理

[`services/SessionMemory/sessionMemory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/services/SessionMemory/sessionMemory.ts) 是真正有价值的后台逻辑：
- 通过 token threshold 和 tool-call threshold 决定是否值得抽取
- 会避免在最后一轮还有 tool calls 时贸然抽取
- `setupSessionMemoryFile()` 通过 `FileReadTool` 读取现有 memory，而不是偷偷自己绕过工具体系
- 真正的抽取由 forked subagent 在后台做，不阻塞主流程

这一点很关键。Claude Code 不是“每轮都总结”，而是把 memory 提取当成一种节奏受控的后台工作。

## 为什么这样做
如果 memory 每轮都更新，系统会变慢、变吵、变得不可控。
如果 memory 完全手动，长任务又会丢失真正重要的上下文。

Claude Code 的做法是：
- 让 `/memory` 保留人工控制入口
- 让 session memory 在后台按阈值抽取
- 让抽取动作本身也复用 agent 能力

这是比“直接接个向量库”更产品化的设计。

## 本章关键文件
- [memory.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/memory.tsx)
- [sessionMemory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/services/SessionMemory/sessionMemory.ts)
- `services/extractMemories/*`
- `services/teamMemorySync/*`

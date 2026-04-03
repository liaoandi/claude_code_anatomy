# 第 10 章 Memory 功能

## 这个功能是什么

Claude Code 的 memory 不是简单"存一下"，而是把当前会话里真正值得保留的信息沉淀成更长期的材料。

这一章讲长期记忆与后台提取。Session 续接和当前窗口压缩在后续两章分别讲。

## 用户如何感知它

用户会感知到三种层次：
- `/memory` 打开或编辑 memory 文件，可以直接手工增删
- 某些对话自动沉淀到 session memory，下次会话能"记住"关键信息
- 某些内容可能进一步进入 team memory 或更长期的同步流程

## 实现链路

memory 的链路分前台和后台两段：

**前台入口**：用户通过 `/memory` 直接打开 memory 文件，在外部编辑器里手工编辑。

**后台提取链路**：
1. 系统监控 token 消耗量和工具调用次数
2. 当 token threshold 或 tool-call threshold 触发时，决定是否值得做一次提取
3. 避免在最后一轮还有活跃 tool calls 时贸然提取（防止打断执行流）
4. 启动 forked subagent 在后台执行提取，不阻塞主流程
5. 提取结果写回 markdown memory 文件

## 关键源码点

[`commands/memory/memory.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/memory.tsx) 是前台入口：
- 先 `clearMemoryFileCaches()` 再 `getMemoryFiles()`，避免缓存过期导致初次打开闪烁
- 打开时确保目录和文件存在（首次使用自动创建）
- 把 memory 文件交给外部编辑器处理，不在 UI 里内嵌编辑

[`services/SessionMemory/sessionMemory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/services/SessionMemory/sessionMemory.ts) 是真正有价值的后台逻辑：

**触发条件**：不是每轮都提取，而是看两个阈值：
- token threshold：当前会话消耗的 token 超过阈值时触发评估
- tool-call threshold：工具调用次数达到阈值时触发评估

两个阈值都是为了确保"这次会话发生了足够多的事情，提取才有意义"。空会话或极短会话不会触发后台提取。

**提取时机控制**：
- 避免在最后一轮还有 tool calls 时提取（会话还没结束，提取内容不完整）
- 提取是异步的，通过 forked subagent 执行，不阻塞主对话

**复用工具体系**：`setupSessionMemoryFile()` 通过 `FileReadTool` 读取现有 memory，而不是绕过工具体系直接读文件。这意味着读取操作仍然经过权限检查和系统侧观测，保持一致性。

`services/extractMemories/*` 是具体的提取逻辑所在——提取本身是由 subagent 完成的，subagent 有自己的上下文，可以做摘要和结构化，不是简单地截取片段。

## 为什么这样做

**为什么不每轮都更新 memory？**
频繁更新会让系统变慢、变吵、引入噪声。大多数对话轮次不产生值得长期保留的信息，强制提取会稀释 memory 的质量。

**为什么不完全手动？**
长任务会话里，真正重要的信息往往在第 20 轮、第 30 轮才出现。让用户全程手工维护 memory 不现实。

**为什么提取用 subagent？**
普通字符串处理很难判断"哪些内容值得保留、以什么格式保留"。用 subagent 做提取，可以做真正的语义理解和结构化，不是简单截取文本。这是"比直接接向量库更产品化的设计"——向量库解决的是检索问题，memory 提取解决的是"什么值得记"的问题。

## 本章关键文件
- [memory.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/memory/memory.tsx)
- [sessionMemory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/services/SessionMemory/sessionMemory.ts)
- `services/extractMemories/*`
- `services/teamMemorySync/*`

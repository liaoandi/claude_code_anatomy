# 第 11 章 Session、Resume、History

## 这个功能是什么

Claude Code 不是一次性问答器，它必须处理历史输入、会话恢复和长任务续接。

这一章讲"会话如何续上"。长期 memory 提取在上一章，当前窗口压缩在下一章。

## 用户如何感知它

重度用户依赖三类能力：
- 上下方向键或 picker 里的历史输入（输入过的内容可以复用）
- 恢复之前的线程（不同 session 可以继续）
- 在 remote/viewer 场景里继续往前翻更早的会话消息

## 实现链路

这里有两条独立链路：

**本地历史链路**：把输入和 paste 引用写到全局 `history.jsonl`，倒序读取，支持跨 session 复用。

**远程会话链路**：通过 session events API 分页加载更老消息，支持 viewer 模式和 remote 场景下的历史滚动。

## 关键源码点

[`history.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/history.ts) 负责本地历史总账：
- 历史存成 `history.jsonl`，append-only，不做删改
- 通过 `makeLogEntryReader()` 倒序读取，最近的优先返回
- 支持 pasted text / image 引用的格式化和展开（粘贴的大段内容不原文存储，存引用）
- `getHistory()` 优先返回当前 session 的历史，再补其他 session 的历史，当前上下文优先

[`assistant/sessionHistory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/assistant/sessionHistory.ts) 是远程事件分页层：
- 先创建带 OAuth headers 的 `HistoryAuthCtx`
- 再按 `anchor_to_latest`（初次加载最新页）或 `before_id`（往前翻页）分页拉取事件

[`hooks/useAssistantHistory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/hooks/useAssistantHistory.ts) 把远程历史接回 UI：
- 初次挂载先拉最新页
- 往上滚动时按阈值触发拉取更老页（类似社交 feed 的懒加载）
- 通过 scroll anchor 保持 prepend 后视口不跳（消息加在上方但不影响当前阅读位置）
- 用 sentinel message 表示 `loading`、`failed`、`start of session` 等边界状态

## 为什么这样做

Claude Code 显然不相信"把所有历史全塞在当前上下文里"这条路。它把 history 拆成三块：
- **本地输入历史**：轻量，只存输入，支持快速回填
- **远程事件分页**：完整会话记录，按需加载，不一次性拉取
- **UI 里的懒加载续接**：scroll anchor 保证体验，不因为加载历史导致界面跳动

这样做的好处是：
- 长会话不会一次性拖垮界面渲染
- 恢复能力不依赖模型记忆，是独立的持久化机制
- remote viewer 场景也能平滑回放历史，不需要重跑会话

## 本章关键文件
- [history.ts](/Users/antonio/Desktop/cc2.1.88/all/src/history.ts)
- [sessionHistory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/assistant/sessionHistory.ts)
- [useAssistantHistory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/hooks/useAssistantHistory.ts)
- `commands/resume/*`

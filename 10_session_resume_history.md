# 第 10 章 Session、Resume、History

## 这个功能是什么
Claude Code 不是一次性问答器，所以它必须处理历史输入、会话恢复和长任务续接。

这一章讲“会话如何续上”，不展开长期 memory 提取和当前窗口压缩。

## 用户如何感知它
重度用户会依赖三类能力：
- 上下方向键或 picker 里的历史输入
- 恢复之前的线程
- 在 remote/viewer 场景里继续往前翻更早的会话消息

## 实现链路
这一块实际上有两条链路：
1. 本地历史链路：把输入和 paste 引用写到全局 `history.jsonl`
2. 远程会话链路：通过 session events API 分页加载更老消息

## 关键源码点
[`history.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/history.ts) 负责的是“本地历史总账”：
- 历史存成 `history.jsonl`
- 通过 `makeLogEntryReader()` 倒序读取
- 支持 pasted text / image 引用的格式化和展开
- `getHistory()` 会优先返回当前 session 的历史，再补别的 session

[`assistant/sessionHistory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/assistant/sessionHistory.ts) 则是远程事件分页层：
- 先创建带 OAuth headers 的 `HistoryAuthCtx`
- 再按 `anchor_to_latest` 或 `before_id` 分页拉取事件

[`hooks/useAssistantHistory.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/hooks/useAssistantHistory.ts) 把这套远程历史真正接回 UI：
- 初次挂载先拉最新页
- 往上滚时再按阈值继续拉更老页
- 通过 scroll anchor 保持 prepend 后视口不跳
- 用 sentinel message 表示 `loading`、`failed`、`start of session`

## 为什么这样做
Claude Code 显然不相信“把所有历史全塞在当前上下文里”这条路。它把 history 拆成：
- 本地输入历史
- 远程事件分页
- UI 里的懒加载续接

这样做的好处是：
- 长会话不会一次性拖垮界面
- 恢复能力不依赖模型记忆
- 远程 viewer 场景也能平滑回放历史

## 本章关键文件
- [history.ts](/Users/antonio/Desktop/cc2.1.88/all/src/history.ts)
- [sessionHistory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/assistant/sessionHistory.ts)
- [useAssistantHistory.ts](/Users/antonio/Desktop/cc2.1.88/all/src/hooks/useAssistantHistory.ts)
- `commands/resume/*`

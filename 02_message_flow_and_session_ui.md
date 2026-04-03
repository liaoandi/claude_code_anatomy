# 第 2 章 消息流与会话界面

## 这个功能是什么
Claude Code 的会话界面不是普通聊天记录，而是一条统一的消息流。工具调用、状态提示、失败、拒绝、diff、系统信息，最后都要落成 message。

所以消息在 Claude Code 里不是 UI 装饰，而是系统的统一展示协议。

## 用户如何感知它
用户在界面里会持续看到：
- assistant 文本
- tool use 与 tool result
- permission rejection
- diff 与状态提示
- 后台任务和系统通知

这就是为什么 Claude Code 看起来更像任务控制台，而不只是对话框。

## 实现链路
一条典型链路是：
1. 工具层执行某个动作
2. 工具通过 `renderToolUseMessage`、`renderToolResultMessage`、`renderToolUseErrorMessage` 产出可显示消息
3. `Messages.tsx` 把原始消息做 normalize、reorder、grouping 和 collapse
4. `MessageRow`、`Message`、`StatusNotices` 等组件再把它渲染为终端 UI

重点在第 2 步。很多工具不是“先返回数据，再让 UI 猜怎么画”，而是工具协议天然就带显示方式。

## 关键源码点
[`components/Messages.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/Messages.tsx) 透露出几个强信号：
- 消息会先经历 `normalizeMessages`、`reorderMessagesInUI`、`applyGrouping`
- 背景 bash 通知、read/search 结果、hook summaries 都会被 collapse
- Brief 模式还有专门的过滤逻辑 `filterForBriefTool` 和 `dropTextInBriefTurns`
- Logo、StatusNotices 这些顶部区域也被视为消息流体验的一部分

这说明 Claude Code 的 message 体系不仅负责显示，还负责“把执行轨迹整理成可读结构”。

## 为什么这样做
如果工具结果不统一纳入消息流，系统会出现两个问题：
- 用户不知道 agent 刚才做了什么
- 每类功能都得自己发明一套独立 UI 状态

Claude Code 选择把 message 作为统一展示层，收益很直接：
- 会话可回放
- 状态可审计
- 工具、权限、diff、通知都能复用同一套容器

## 本章关键文件
- [Messages.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/Messages.tsx)
- [Message.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/Message.tsx)
- [MessageRow.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/MessageRow.tsx)

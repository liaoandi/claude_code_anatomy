# 第 2 章 消息流与会话界面

## 这个功能是什么

Claude Code 的会话界面不是普通聊天记录，而是一条统一的消息流。工具调用、状态提示、失败、拒绝、diff、系统通知，全部落成 message 才能显示。

所以消息在 Claude Code 里不是 UI 装饰，而是系统的统一展示协议。工具层通过 `renderTool*Message` 把自己的输出挂进来，UI 层再统一处理和渲染。两者之间有一套明确的协议边界。

## 用户如何感知它

用户在界面里会持续看到：
- assistant 文本回复
- tool use 调用记录与 tool result 结果
- permission rejection 及重试入口
- diff 与文件变更提示
- 后台任务通知和系统信息

这就是为什么 Claude Code 看起来更像任务控制台，而不只是一个对话框。

## 实现链路

一条典型的消息生命周期是：

1. 工具层执行某个动作
2. 工具通过 `renderToolUseMessage`、`renderToolResultMessage`、`renderToolUseErrorMessage` 生成可显示消息
3. `Messages.tsx` 把原始消息做 normalize、reorder、grouping 和 collapse
4. `MessageRow`、`Message`、`StatusNotices` 等组件把它渲染为终端 UI

重点在第 2 步：很多工具不是"先返回数据，再让 UI 猜怎么画"，而是工具协议天然就包含显示方式。工具和消息层的耦合是刻意设计的，不是实现细节。

## 关键源码点

[`components/Messages.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/Messages.tsx) 透露出几个关键信号：

- 消息先经历 `normalizeMessages`（格式统一）、`reorderMessagesInUI`（调整顺序）、`applyGrouping`（分组）
- 后台 bash 通知、read/search 结果、hook summaries 默认 collapse，减少干扰
- Brief 模式有专门的过滤逻辑 `filterForBriefTool` 和 `dropTextInBriefTurns`，只保留关键输出
- Logo、StatusNotices 这类顶部区域也被视为消息流体验的一部分，统一管理

这说明消息层不只负责显示，还负责"把执行轨迹整理成可读结构"——原始消息和用户看到的消息之间有非平凡的处理过程。

## 为什么这样做

如果工具结果不统一纳入消息流，系统会出现两个问题：
- 用户不知道 agent 刚才做了什么，执行轨迹不可见
- 每类功能都得自己发明一套独立 UI 状态，维护成本很高

Claude Code 把 message 作为统一展示层，直接带来三个好处：
- **会话可回放**：所有执行历史都在消息流里，不依赖额外日志
- **状态可审计**：权限拒绝、工具调用、diff 都有对应消息记录
- **工具复用展示容器**：每个工具不用自己搭 UI，只需实现 `renderTool*Message`

## 本章关键文件
- [Messages.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/Messages.tsx)
- [Message.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/Message.tsx)
- [MessageRow.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/MessageRow.tsx)

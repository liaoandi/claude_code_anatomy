# 第 12 章 Context 管理

## 这个功能是什么

Context 管理解决的是"当前窗口里什么该留、什么该压缩、用户怎么看到这件事"。

这一章讲当前上下文预算控制。历史续接在上一章，长期 memory 沉淀在第 10 章。

## 用户如何感知它

用户会在这些地方直接看到 context 管理：
- `/compact` 手动压缩当前上下文
- `/context` 查看当前上下文使用情况
- context visualization 界面展示"上下文花在哪里"
- 某些对话中自动发生的 compact 或 collapse，输出被折叠

## 实现链路

上下文管理有三层：

**命令层**：允许用户主动查询（`/context`）和触发压缩（`/compact`）。

**服务层**：`services/compact/*` 负责自动 compact 的时机和算法：
1. 监控当前 token 使用量
2. 当接近模型上下文窗口上限时触发自动 compact 评估
3. compact 算法选择要保留的内容：优先保留最近几轮、工具结果的关键部分、用户明确引用的内容
4. 被压缩的内容用 summary 替代，打上 `summarized` 标记
5. `context collapse` 是更激进的压缩，把多轮内容合并成单个摘要块

**可视化层**：把"上下文花在哪里"呈现给用户，而不是让 compact 变成黑盒。

## 关键源码点

[`components/ContextVisualization.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/ContextVisualization.tsx) 把通常藏在系统内部的 token 分配做成了显式产品界面：
- 按 category 展示 context usage（model、memory files、MCP tools、skills、agents、message breakdown）
- 如果启用了 context collapse，通过 `CollapseStatus` 告诉用户有多少内容被 summarized / staged
- 让用户能看到"为什么上下文满了"，而不是突然发现系统忘了之前说的话

**compact 算法的核心判断**（`services/compact/*`）：
- **保留最近**：最近 N 轮对话默认完整保留，不压缩
- **保留工具结果摘要**：工具调用结果通常很长，compact 后只保留关键输出
- **保留用户明确引用的内容**：如果某段内容被后续对话引用，不压缩
- **合并相似轮次**：多轮纯文本对话可以合并成摘要，减少 token 消耗

**context collapse** 比 compact 更激进：把多轮内容（包括工具调用链）折叠成一个 summary block，通过 `CollapseStatus` 标注折叠了多少内容。适合长任务的中间阶段，在新阶段开始前清理历史积累。

## 为什么这样做

长任务里最难受的体验之一，就是用户不知道系统为什么突然"忘了"什么、压了什么、保留了什么。

Claude Code 把 context 管理做成显式功能，有三个好处：
- **降低黑箱感**：ContextVisualization 让 token 分配透明，用户知道上下文去哪了
- **让压缩行为可解释**：summarized 和 staged 的标记让用户能看到"什么被压缩了"
- **让高级用户能主动管理预算**：`/compact`、`/context` 给了显式控制入口，不是完全被动接受

把 context 预算做成可见化 UI，本质上是在降低用户对 AI 行为的不确定感。

## 本章关键文件
- [ContextVisualization.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/ContextVisualization.tsx)
- `commands/compact/*`
- `commands/context/*`
- `services/compact/*`

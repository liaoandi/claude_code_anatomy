# 第 5 章 改文件与 Diff 展示

## 这个功能是什么
改文件是 Claude Code 最核心的执行能力之一。但它不是“随便写磁盘”，而是“在权限、校验和可审阅前提下修改文件”。

Diff 展示则是这项能力的配套界面。没有 diff，文件编辑就失去可见性和信任基础。

## 用户如何感知它
用户会明显感知到：
- Claude Code 会尝试精准替换，而不是盲改整文件
- 修改前后通常能看到 diff
- 某些编辑会被拒绝或要求确认
- 文件被外部改动时，编辑可能失败而不是强行覆盖

## 实现链路
一条编辑链路通常是：
1. 模型调用 `FileEditTool`
2. 工具先做 `file_path` 规范化和写权限检查
3. 再校验 `old_string` / `new_string`、文件大小、文件是否存在、是否被外部改动
4. 通过后生成 patch 或实际写回内容
5. 再把 diff 和结果通过统一消息机制显示给用户

## 关键源码点
[`tools/FileEditTool/FileEditTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileEditTool/FileEditTool.ts) 暴露出 Claude Code 对编辑能力的强约束：
- `strict: true`，说明这是个不鼓励模糊输入的工具
- 会先 `expandPath`，避免路径别名绕过校验
- 会检查 deny rule 和 write permission
- 会拦截 team memory secrets 污染
- 会限制 1 GiB 以上文件，防止 OOM
- 会检测文件不存在、文件已变更、空文件创建等细分场景
- 集成了 file history、git diff、LSP diagnostics 和 VS Code 通知

这说明 FileEditTool 不是单一写文件动作，而是一整套“安全编辑管线”。

## 为什么这样做
如果编辑只依赖 shell，Claude Code 在产品层会失去两样最关键的东西：
- 可控的修改边界
- 可读的结果展示

所以 Claude Code 把编辑能力单独做成一级工具。这样做的直接好处是：
- 可以对编辑前提做细粒度校验
- 可以稳定生成 diff
- 可以把拒绝、冲突、失败都纳入统一用户体验

## 本章关键文件
- [FileEditTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileEditTool/FileEditTool.ts)
- `tools/FileWriteTool/*`
- `components/StructuredDiff*`
- `components/FileEditToolDiff.tsx`

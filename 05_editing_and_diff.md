# 第 5 章 改文件与 Diff 展示

## 这个功能是什么

改文件是 Claude Code 最核心的执行能力之一。但它不是"随便写磁盘"，而是"在权限、校验和可审阅前提下修改文件"。

Diff 展示是这项能力的配套界面。没有 diff，文件编辑就失去可见性和信任基础——用户不知道系统改了什么，也没有办法判断改得对不对。

## 用户如何感知它

用户会明显感知到：
- Claude Code 会做精准替换，而不是盲改整文件
- 修改前后能看到 diff，知道具体改了哪些行
- 某些编辑会被拒绝或要求确认
- 文件被外部改动时，编辑可能失败而不是强行覆盖

## 实现链路

一条编辑链路通常是：

1. 模型调用 `FileEditTool`，传入 `file_path`、`old_string`、`new_string`
2. 工具做 `file_path` 规范化和写权限检查
3. 校验 `old_string` 在文件里是否唯一（防止误改）、文件大小、文件是否存在、是否被外部改动
4. 通过后生成 patch，写回文件
5. 把 diff 和结果通过统一消息机制显示给用户

`FileWriteTool` 走类似链路，区别是整文件覆盖而不是精准替换，适用于新建文件或完全重写的场景。

## 关键源码点

[`tools/FileEditTool/FileEditTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileEditTool/FileEditTool.ts) 暴露出 Claude Code 对编辑能力的强约束：

- `strict: true`：输入 schema 不允许模糊参数，每个字段都有明确类型
- `expandPath`：先展开路径别名，防止路径别名绕过后续校验
- deny rule 和 write permission 检查：基于用户配置的权限规则决策
- team memory secrets 拦截：防止把敏感内容写进共享 memory
- 1 GiB 以上文件拦截：防止大文件写入导致 OOM
- 文件不存在、文件已被外部改变、空文件创建等细分场景各有专门处理
- 集成了 file history、git diff、LSP diagnostics 和 VS Code 通知

这说明 FileEditTool 不是单一写文件动作，而是一整套"安全编辑管线"。

Diff 展示方面，`components/StructuredDiff*` 和 `components/FileEditToolDiff.tsx` 负责把 patch 结果渲染成用户可读的 diff 界面，直接显示在消息流里，不需要用户自己运行 `git diff`。

## 为什么这样做

如果编辑只依赖 shell，Claude Code 在产品层会失去两样最关键的东西：

**可控的修改边界**：裸 `fs.write` 无法做前置校验，改错了也不知道。FileEditTool 要求 `old_string` 在文件里必须存在且唯一，从根本上防止了"改错地方"。

**可读的结果展示**：shell 写文件不会自动生成 diff。没有 diff，用户要自己去对比前后差异，信任成本很高。

把编辑能力做成一级工具的直接好处：
- 细粒度前置校验，减少误操作
- 稳定生成 diff，让修改透明可见
- 拒绝、冲突、失败都进统一用户体验，不是悄悄失败

## 本章关键文件
- [FileEditTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileEditTool/FileEditTool.ts)
- `tools/FileWriteTool/*`
- `components/StructuredDiff*`
- `components/FileEditToolDiff.tsx`

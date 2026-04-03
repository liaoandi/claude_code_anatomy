# 第 4 章 读文件、搜文件、看上下文

## 这个功能是什么

Claude Code 要先读懂项目，才能改项目。"读文件、搜文件、按范围取内容"是最基础的一层能力。

这层能力的重点不是把文件内容返回给模型，而是把"安全、成本、可读性"一起处理掉。有三个主要工具：FileReadTool（读内容）、GrepTool（按模式搜索）、GlobTool（按路径匹配文件）。

## 用户如何感知它

用户通常感知到的是：
- agent 会先读某些文件再开始改
- 长文件不会一次性整块塞进上下文
- 搜索和读取是不同动作，适用于不同场景
- 某些文件会带行号、范围或格式化处理

## 实现链路

**FileReadTool** 的典型链路：
1. 模型调用工具，传入文件路径（可选 offset/limit）
2. 工具做路径规范化、权限检查、设备文件拦截、大文件 token 预算判断
3. 根据文件类型走不同读取分支：notebook、PDF、图片、普通文本
4. 把结果包装成统一 tool result，附上 UI 可展示信息

**GrepTool** 的典型链路：
1. 模型调用工具，传入 pattern、路径（可选文件类型过滤）
2. 工具通过 ripgrep 执行搜索，处理结果格式化和行号标注
3. 结果按相关性截断（避免海量匹配把上下文打爆）
4. 返回结构化匹配列表，带文件路径和行号

**GlobTool** 的典型链路：
1. 模型调用工具，传入 glob pattern（如 `**/*.ts`、`src/**/index.*`）
2. 工具展开匹配，过滤掉 `.gitignore` 覆盖的路径和系统文件
3. 按修改时间排序返回文件列表
4. 结果用于指引后续 FileReadTool 读取，两者常配合使用

## 关键源码点

[`tools/FileReadTool/FileReadTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileReadTool/FileReadTool.ts) 说明 Claude Code 的"读文件"远比看起来复杂：
- 会屏蔽 `/dev/zero`、`/dev/random`、`/dev/tty` 这类危险路径
- 有读权限检查，不是所有文件都能读
- 根据 token 数限制决定读整文件还是建议 offset/limit 分段
- 能识别 session memory、session transcript 等特殊文件类型
- 能根据文件类型走 PDF、notebook、图片等不同处理分支
- 通过 `registerFileReadListener` 把读文件事件暴露给其他服务消费

`tools/GrepTool/*` 说明搜索不是裸 grep：
- 底层用 ripgrep，搜索速度和 `.gitignore` 处理明显优于系统 grep
- 会对结果做截断和格式化，防止海量匹配撑爆上下文
- 返回结构化结果（文件名 + 行号 + 匹配内容），而不是原始文本流

`tools/GlobTool/*` 说明文件发现不是 `ls -R`：
- 支持完整 glob 语法（`**`、`{}`、`?`）
- 自动尊重 `.gitignore` 规则，不返回构建产物或依赖目录
- 按修改时间排序，帮助模型优先关注最近改动的文件

## 为什么这样做

**为什么不直接用 shell 的 `cat`、`grep`、`find`？**

如果读取逻辑只是 shell 命令的外壳，Claude Code 很快会遇到：
- 超大文件把上下文打爆（`cat` 没有 token 预算概念）
- 特殊设备文件把进程卡死（`/dev/zero` 读取会无限阻塞）
- 图片、PDF、notebook 无法统一进入主循环
- 搜索结果无限制返回，每次都要手动截断

**为什么搜索和读取是两个工具，而不是一个？**

GrepTool 和 GlobTool 的职责不同：
- Grep 是"找内容"——知道要找什么，不知道在哪
- Glob 是"找文件"——知道文件结构，要获取候选列表
- FileRead 是"取内容"——知道具体文件，要读进来

分开工具让模型调用意图更清晰，权限也可以独立控制（只读文件名 vs 读文件内容是不同的权限粒度）。

## 本章关键文件
- [FileReadTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileReadTool/FileReadTool.ts)
- `tools/GrepTool/*`
- `tools/GlobTool/*`

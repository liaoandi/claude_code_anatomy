# 第 4 章 读文件、搜文件、看上下文

## 这个功能是什么
Claude Code 要先读懂项目，才能改项目。所以“读文件、搜文件、按范围取内容”是最基础的一层能力。

这层能力的重点不是简单把文件内容返回给模型，而是把“安全、成本、可读性”都一起处理掉。

## 用户如何感知它
用户通常感知到的是：
- agent 会先读某些文件
- 长文件不会一次性整块塞进上下文
- 搜索和读取是不同动作
- 某些文件会带行号、范围或格式化处理

## 实现链路
读文件的一条典型链路是：
1. 模型调用 `FileReadTool`
2. 工具先做路径规范化、权限检查、设备文件拦截和大文件 token 预算判断
3. 再根据文件类型走不同读取逻辑，例如 notebook、PDF、图片、普通文本
4. 最后把结果包装成统一 tool result，并附上 UI 层可展示信息

## 关键源码点
[`tools/FileReadTool/FileReadTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileReadTool/FileReadTool.ts) 说明 Claude Code 的“读文件”远比看起来复杂：
- 会屏蔽 `/dev/zero`、`/dev/random`、`/dev/tty` 这类危险路径
- 会做 read permission 检查
- 会根据 token 数限制读整文件还是建议 offset/limit
- 能识别 session memory、session transcript 这类特殊文件
- 能根据文件类型走 PDF、notebook、图片等不同分支
- 还有 `registerFileReadListener`，说明读文件事件会被别的服务消费

这说明 FileReadTool 实际上兼顾了三件事：
- 安全读取
- 语料整形
- 系统侧观测

## 为什么这样做
如果“读文件”只是 `fs.readFile` 的外壳，Claude Code 很快就会遇到：
- 超大文件把上下文打爆
- 特殊设备文件把进程卡死
- 图片、PDF、notebook 无法统一进入主循环

Claude Code 选择把读取逻辑产品化，而不是把它留给 shell 命令临时解决。这让读取能力可控、可审计，也更容易被模型稳定调用。

## 本章关键文件
- [FileReadTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/FileReadTool/FileReadTool.ts)
- `tools/GrepTool/*`
- `tools/GlobTool/*`

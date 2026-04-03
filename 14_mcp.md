# 第 14 章 MCP

## 这个功能是什么
MCP 是 Claude Code 的协议级扩展框架。它让核心系统不用提前认识所有外部工具，也能在运行时接入它们。

## 用户如何感知它
用户感知 MCP 的方式通常是：
- 接入某个 MCP server
- Claude Code 获得一批新的 tool 或 resource
- 这些能力在体验上看起来几乎像内建功能

## 实现链路
MCP 的典型链路是：
1. 连接 MCP server
2. 运行时发现服务器暴露的 tool / resource
3. 用 `MCPTool` 这种模板工具承接调用
4. 再由 list/read resource 等专门工具把资源显式纳入系统

## 关键源码点
[`tools/MCPTool/MCPTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/MCPTool/MCPTool.ts) 非常说明问题：
- `isMcp: true`
- `inputSchema` 是 `z.object({}).passthrough()`，允许 open-world 输入
- `name`、`description`、`prompt`、`call` 都会被 mcp client 在运行时覆盖
- 权限行为是 `passthrough`

换句话说，`MCPTool` 不是最终工具，而是“可被外部 server 具体化的工具模板”。

这正是协议级扩展和内建工具的最大区别。

## 为什么这样做
Claude Code 明显不想把所有第三方系统都硬编码进核心仓库。
MCP 给它带来的不是又一种插件机制，而是更底层的扩展能力：
- 核心只理解协议
- 外部能力在运行时动态出现
- 工具仍然能被纳入统一消息、权限和上下文体系

## 本章关键文件
- [MCPTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/MCPTool/MCPTool.ts)
- `tools/ListMcpResourcesTool/*`
- `tools/ReadMcpResourceTool/*`
- `commands/mcp/*`
- `services/mcp/*`

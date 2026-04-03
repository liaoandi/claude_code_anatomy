# 第 14 章 MCP

## 这个功能是什么

MCP（Model Context Protocol）是 Claude Code 的协议级扩展框架。它让核心系统不用提前认识所有外部工具，也能在运行时接入它们。

和 Web 工具（内建、受控、轻量）、Plugins/Skills（markdown 编译为命令、静态加载）不同，MCP 是真正的运行时动态扩展——工具在连接建立时才出现，核心系统只理解协议。

## 用户如何感知它

用户感知 MCP 的方式通常是：
- 连接某个 MCP server
- Claude Code 获得一批新的 tool 或 resource，看起来几乎像内建功能
- 断开 MCP server 后，这批工具消失

## 实现链路

MCP 的典型链路是：

1. 连接 MCP server（stdio、HTTP SSE 等传输层）
2. 运行时发现服务器暴露的 tools 和 resources
3. 每个 tool 用 `MCPTool` 模板承接，在运行时被具体化
4. resource 通过 `ListMcpResourcesTool` 和 `ReadMcpResourceTool` 显式纳入系统
5. 后续调用和内建工具走同一套权限 + 消息流管道

## 关键源码点

[`tools/MCPTool/MCPTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/MCPTool/MCPTool.ts) 非常说明问题：

- `isMcp: true`：标记来源，便于权限层区分处理
- `inputSchema` 是 `z.object({}).passthrough()`：允许 open-world 输入，因为 MCP server 的参数结构不可预知
- `name`、`description`、`prompt`、`call` 都会被 MCP client 在运行时覆盖：这个类是模板，不是最终工具
- 权限行为是 `passthrough`：权限判断交给 MCP server 侧，不在模板层做

`MCPTool` 不是最终工具，而是"可被外部 server 具体化的工具模板"。这正是协议级扩展和内建工具的最大区别。

**Resource vs Tool 的区分**：
- `ListMcpResourcesTool`：列出 MCP server 暴露的所有 resource（数据，不是动作）
- `ReadMcpResourceTool`：读取具体 resource 内容，纳入上下文
- MCP Tool 调用：触发 MCP server 执行某个动作，有副作用

## 三种扩展方式对比

这里适合把 Web、MCP、Plugins/Skills 放在一起比较：

| 特性 | Web 工具 | MCP Server | Plugins / Skills |
|------|---------|-----------|----------------|
| 扩展方式 | 内建，不可添加 | 运行时动态连接 | 静态目录扫描加载 |
| 工具来源 | Claude Code 核心团队维护 | 任意第三方实现 | markdown 编译 |
| 认证支持 | 不支持 | Server 侧完整支持 | 不涉及 |
| 适合场景 | 抓公开网页内容 | 接入外部系统（DB、API、SaaS） | 工作流复用、自定义命令 |
| 接入成本 | 无需配置，内建 | 中等（实现 MCP 协议） | 最低（写 markdown） |
| 权限粒度 | domain 级 | tool 级（passthrough 到 server） | 命令级 |

## 为什么这样做

Claude Code 明显不想把所有第三方系统都硬编码进核心仓库。MCP 给它带来的不是又一种插件机制，而是更底层的扩展能力：
- 核心只理解协议，不理解具体工具
- 外部能力在运行时动态出现，不需要重新编译或重启
- 工具仍然能被纳入统一消息、权限和上下文体系，不破坏系统一致性

## 本章关键文件
- [MCPTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/MCPTool/MCPTool.ts)
- `tools/ListMcpResourcesTool/*`
- `tools/ReadMcpResourceTool/*`
- `commands/mcp/*`
- `services/mcp/*`

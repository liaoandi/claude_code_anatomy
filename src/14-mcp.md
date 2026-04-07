# 第 14 章 MCP

## 为什么需要一个扩展协议

Claude Code 内建的工具覆盖了文件、shell、web——这已经很强了，但总有它覆盖不到的地方：你的数据库、你们公司的内部 API、特定的 SaaS 工具。

一种解法是：把这些都硬编码进 Claude Code。但这显然不现实，外部系统千变万化，不可能全部内建。

MCP（Model Context Protocol）是另一种解法：定义一套协议，让外部系统自己实现这套协议，然后在运行时接入。Claude Code 只需要理解协议，不需要提前认识每个具体的系统。

## MCP 工具和内建工具有什么不同

内建工具在编译时就确定了——代码里写死了有哪些工具、每个工具能做什么。WebFetchTool 就是典型的内建工具：功能固定，适合抓公开网页，但没有认证能力。

MCP 工具是运行时动态出现的。你连接一个 MCP server，Claude Code 把它声明的能力注册进来，之后就可以调用了。断开连接，这些能力就消失了。需要认证的系统——数据库、内部 API、需要登录的 SaaS——走 MCP 更合适：认证逻辑放在你的 MCP server 里，Claude Code 这边只看到一个普通的 tool 调用。

实现上，`MCPTool` 是一个模板——名字、描述、参数 schema、执行逻辑，都在连接时被 server 返回的信息覆盖。同一个模板，连接不同的 MCP server，就变成不同的工具。

## Tool、Resource、Command 三种能力

说"MCP server 提供工具"其实只说了一半。协议定义了三种能力，区别在于谁来触发、发生在什么阶段：

- **Tool**：模型决定调用。任务执行过程中，模型判断需要做某件事，主动调用 tool，拿到结果再继续。比如查询数据库、发送消息。有参数、有返回值、可能有副作用。
- **Resource**：模型主动读取，用来获取背景信息。不产生副作用，更像一个可以按需读取的文件——比如数据库的 schema、项目文档的目录。风险比 tool 低，权限策略也可以更宽松。
- **Command**：用户触发，不是模型自己决定的。MCP server 暴露的 command 会直接并入 Claude Code 的命令注册表，用户可以像用内建的 `/xxx` 一样调用它。

## 写 MCP Server 的几个关键点

**认证在 server 侧处理**。MCP server 是你自己控制的进程，把认证逻辑放在里面。Claude Code 这边不需要知道你的 API key 或者 OAuth token，它只看到工具调用和返回结果。

**Tool schema 要精确**。MCP 工具的参数 schema 最终决定模型怎么调用这个工具。太宽松（比如只有一个 `query: string`），模型会猜参数格式，结果不稳定。每个字段都应该有明确的类型和描述。

**区分 tool 和 resource**。能用 resource 表达的数据就不要做成 tool。resource 是只读的、无副作用的，权限系统可以给它更宽松的默认策略；tool 有副作用，应该触发确认。把 schema 查询做成 resource 而不是 tool，是一个常见的好实践。

**测试权限边界**。Claude Code 会对 MCP tool 调用做权限判断。测试时不能只测"正常调用成功"，要测"调用被拒绝时 MCP server 怎么响应"，以及"server 返回错误时 Claude Code 怎么展示给用户"。

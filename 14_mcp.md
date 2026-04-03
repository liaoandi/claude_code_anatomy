# 第 14 章 MCP

## 为什么需要一个扩展协议

Claude Code 内建的工具覆盖了文件、shell、web——这已经很强了，但总有它覆盖不到的地方：你的数据库、你们公司的内部 API、特定的 SaaS 工具。

一种解法是：把这些都硬编码进 Claude Code。但这显然不现实，外部系统千变万化，不可能全部内建。

MCP（Model Context Protocol）是另一种解法：定义一套协议，让外部系统自己实现这套协议，然后在运行时接入。Claude Code 只需要理解协议，不需要提前认识每个具体的系统。

## MCP 工具和内建工具有什么不同

内建工具在编译时就确定了——代码里写死了有哪些工具、每个工具能做什么。

MCP 工具是运行时动态出现的。你连接一个 MCP server，它声明自己有哪些 tool 和 resource，Claude Code 把这些注册进来，之后就可以调用了。断开连接，这些工具就消失了。

实现上，`MCPTool` 是一个模板——它的名字、描述、参数 schema、执行逻辑，都在连接时被 MCP client 用 server 返回的信息覆盖。同一个模板，连接不同的 MCP server，就变成不同的工具。

## Tool 和 Resource 的区别

MCP server 可以暴露两种东西：tool 和 resource。

Tool 是动作：执行某个操作，可能有副作用，比如查询数据库、发送消息。

Resource 是数据：只读的内容，比如数据库的 schema、文档库的目录。

这个区分让权限系统可以有不同的策略：读取 resource 通常比执行 tool 风险低，可以有更宽松的默认策略。

## MCP 和内建 Web 工具的分工

一个常见的问题是：WebFetchTool 已经能联网了，为什么还需要 MCP？

WebFetchTool 适合抓公开网页，简单直接，但没有认证能力。

需要认证的系统——数据库、内部 API、需要登录的 SaaS——就得走 MCP。认证逻辑放在你的 MCP server 里，Claude Code 这边只看到一个普通的 tool 调用，不需要知道背后的认证细节。

两者不是替代关系，而是覆盖不同的场景。

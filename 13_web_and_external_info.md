# 第 13 章 Web 与外部信息获取

## 这个功能是什么

Claude Code 不只看本地代码。在需要外部信息时，它通过 Web 工具把网页内容带进主循环。

这一章只讲内建 Web 工具，不展开 MCP 这种协议级扩展。Web 工具是 Claude Code 核心系统的一部分，受同一套权限和消息机制管控；MCP 是外部系统的接入协议，两者层次不同。

## 用户如何感知它

用户会看到 Claude Code：
- 搜索网页并把结果纳入上下文
- 拉取某个 URL 的内容，处理后用于回答或执行任务
- 遇到需要认证的 URL 时，提示应使用专门的 MCP tool

## 实现链路

Web fetch 的典型链路是：

1. 模型调用 `WebFetchTool`，传入 `url` 和 `prompt`（必填）
2. 权限系统按 hostname / domain 做 allow、ask、deny 决策
3. 真正 fetch，把网页内容转成 markdown
4. 把 `prompt` 应用到 fetched content 上，生成结构化结果
5. 结果返回主循环，进入消息流

`WebSearchTool` 类似，区别是先做搜索再拉取，省去用户自己找 URL 的步骤。

## 关键源码点

[`tools/WebFetchTool/WebFetchTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/WebFetchTool/WebFetchTool.ts) 说明 Web 能力在 Claude Code 里不是裸网络请求：

- **输入 schema 同时要求 `url` 和 `prompt`**：强迫调用者明确"取了这段内容要用来做什么"，防止无目的爬取
- **权限决策按 hostname/domain**：不是"能不能联网"的总开关，而是细粒度的域名级控制
- **`isPreapprovedHost`**：某些可信域名（如 docs 站点）预先配置为允许，低摩擦通过
- **`buildSuggestions`**：拦截时给用户提供快速规则建议，让用户能一键添加白名单
- **工具是 `isReadOnly()` 且 `isConcurrencySafe()`**：可以并发调用，不会有副作用冲突
- **prompt 明确警告认证 URL 会失败**：遇到私有系统，提示应使用专门的 MCP tool

这说明 Claude Code 对 Web 能力的定位很清楚：它是"受限的信息提取工具"，不是通用浏览器。

## 为什么这样做

如果把 Web 能力直接塞进模型 backend，系统会丢失两样东西：
- **权限边界**：所有联网行为都绕过了用户可见的控制层
- **用户可见性**：抓了什么内容、从哪里抓的，用户不知道

Claude Code 把 Web 做成工具，让外部信息获取遵守和本地工具一样的规则：
- 能被展示（消息流里有 fetch 记录）
- 能被批准或拒绝（domain 级权限）
- 能被统计和审计（token 计入 context 预算）

**和 MCP 的分工**：WebFetchTool 适合"抓公开网页内容"；如果要接入需要认证的 API 或专有系统，应该用 MCP Server——认证逻辑放在 MCP Server 里，而不是让 WebFetchTool 处理。

## 本章关键文件
- [WebFetchTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/WebFetchTool/WebFetchTool.ts)
- `tools/WebSearchTool/*`

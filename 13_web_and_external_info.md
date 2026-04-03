# 第 13 章 Web 与外部信息获取

## 这个功能是什么
Claude Code 并不只看本地代码。在需要外部信息时，它会通过 Web 工具把网页内容带进主循环。

这一章只讲内建 Web 工具，不展开 MCP 这种协议级扩展。

## 用户如何感知它
用户会看到 Claude Code：
- 搜索网页
- 拉取某个 URL 的内容
- 把抓到的内容经过处理后再用于回答或执行任务

## 实现链路
Web fetch 的典型链路是：
1. 模型调用 `WebFetchTool`
2. 工具 schema 收到 `url` 和 `prompt`
3. 权限系统先按 hostname / domain 做 allow、ask、deny 决策
4. 真正 fetch 后把网页转成 markdown 内容
5. 再把 prompt 应用到 fetched content 上，生成结构化结果返回主循环

## 关键源码点
[`tools/WebFetchTool/WebFetchTool.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/tools/WebFetchTool/WebFetchTool.ts) 说明 Web 能力在 Claude Code 里不是裸网络请求：
- 输入 schema 同时要求 `url` 和 `prompt`
- 权限决策不是按“能不能联网”，而是按具体 hostname/domain 规则
- 有 `isPreapprovedHost` 和 `buildSuggestions`，说明某些域名可以低摩擦通过
- 工具本身是 `isReadOnly()` 且 `isConcurrencySafe()`
- prompt 明确警告 authenticated/private URL 会失败，提示应优先找 specialized MCP tool

这说明 Claude Code 对 Web 能力的定位很清楚：
它是“受限的信息提取工具”，不是通用浏览器。

## 为什么这样做
如果把 Web 能力直接塞进模型 backend，系统会丢失两样东西：
- 权限边界
- 用户可见性

Claude Code 把 Web 做成工具，是为了让外部信息获取也遵守和本地工具一样的规则：
- 能被展示
- 能被批准或拒绝
- 能被统计和审计

## 本章关键文件
- [WebFetchTool.ts](/Users/antonio/Desktop/cc2.1.88/all/src/tools/WebFetchTool/WebFetchTool.ts)
- `tools/WebSearchTool/*`

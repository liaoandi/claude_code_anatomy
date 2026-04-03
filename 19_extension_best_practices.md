# 第 19 章 扩展设计最佳实践

这一章是全书的总结性章节，面向想扩展 Claude Code 能力的读者。前 18 章讲的是"系统怎么做"，这一章讲"你应该怎么做"。

## 三种扩展方式及其适用场景

Claude Code 提供了三种不同层次的扩展机制，适合不同类型的需求：

| 扩展方式 | 适合场景 | 接入成本 | 能力边界 |
|---------|---------|---------|---------|
| **Skill / Plugin** | 工作流复用、自定义 slash command、提示词封装 | 最低，写 markdown | 在现有工具能力范围内编排 |
| **MCP Server** | 接入外部系统（数据库、API、SaaS）、动态暴露工具 | 中等，实现 MCP 协议 | 可以暴露全新工具和资源 |
| **内建工具（源码级）** | 需要深度集成权限/消息/任务体系的能力 | 最高，需修改核心代码 | 完全控制，能融入所有系统层 |

**如何选择**：
- 如果你只是想封装一段常用工作流或 prompt，选 Skill。
- 如果你要接入一个外部系统（如数据库、Slack、JIRA），选 MCP Server。
- 如果你的能力需要权限判断、自定义 diff 展示或深度任务集成，选内建工具（或基于 MCP 做轻量适配）。

## 写一个内建工具的标准结构

每个内建工具需要实现以下接口，缺一不可：

```typescript
// 1. 工具元信息
name: string                    // 唯一标识，用于命令注册和消息显示
description: string             // 模型看到的工具说明，决定何时调用
inputSchema: ZodSchema          // 输入参数定义，strict: true 防止模糊输入

// 2. 权限检查（每次调用前触发）
checkPermissions(input, context): PermissionResult
// 返回 allow / ask / deny，配合 ApprovalUI 使用

// 3. 执行逻辑
call(input, context): ToolResult
// 执行实际操作，失败时抛出结构化错误，不要静默吞掉

// 4. 展示协议（消息层的一部分）
renderToolUseMessage(input): ReactNode    // 工具被调用时显示什么
renderToolResultMessage(result): ReactNode // 工具执行结果显示什么
renderToolUseErrorMessage(error): ReactNode // 失败时显示什么
```

**几个常见错误**：
- `inputSchema` 设置 `passthrough()`：除非是 MCP 工具模板，否则不要这样做。它会让模型传入任意参数，绕过类型安全。
- `checkPermissions` 只做空实现：每个写操作或网络操作都应该有明确的权限策略。
- `renderToolResultMessage` 返回 null：工具结果必须可见，不要让系统无声执行。

## 权限接入的正确姿势

权限系统不是总开关，而是按调用上下文做细粒度决策。正确的做法是：

```typescript
checkPermissions(input, context) {
  // 1. 先查 deny rule：如果明确被拒绝，直接 deny
  if (isDenied(input.file_path, context.rules)) return { result: 'deny' }

  // 2. 再查 allow rule：如果已明确授权，直接 allow
  if (isAllowed(input.file_path, context.rules)) return { result: 'allow' }

  // 3. 没有匹配规则时，进入 ask（触发 ApprovalUI）
  return {
    result: 'ask',
    message: `需要写入 ${input.file_path}`,
    suggestions: buildSuggestions(input)  // 给用户提供快速规则建议
  }
}
```

**权限规则示例**：
```
allow:Bash(git *:*)         # 允许所有 git 命令
allow:FileEdit(/src/**:*)   # 允许编辑 src 下所有文件
deny:Bash(rm -rf *:*)       # 拒绝危险删除命令
ask:WebFetch(*.external.com:*) # 外部域名需要每次确认
```

规则按 `deny > allow > ask` 优先级匹配，第一条命中的规则生效。

## 消息展示的最佳实践

工具的展示质量直接影响用户对系统的信任感。几个建议：

**展示要对称**：`renderToolUseMessage` 和 `renderToolResultMessage` 应该成对设计。用户看到"准备做什么"，也要看到"做完了什么"。

**失败要可读**：`renderToolUseErrorMessage` 不要只返回原始错误堆栈。把失败原因翻译成用户能理解的语言，并给出下一步建议。

**可折叠的内容折叠**：搜索结果、读取内容这类大量重复输出，应该默认 collapse。参考 `Messages.tsx` 里的 `filterForBriefTool` 模式。

**不要自己维护 UI 状态**：所有输出都通过消息系统，不要在工具里维护独立的 React 状态。工具是无状态的执行单元，状态由 provider 层管理。

## 长任务要接入任务体系

任何执行时间超过几秒的操作，都应该注册为 Task，而不是在 `call()` 里同步阻塞：

```typescript
// 错误做法：直接同步执行
async call(input) {
  const result = await longRunningOperation(input) // 阻塞主线程
  return result
}

// 正确做法：注册为 Task
async call(input, context) {
  const task = await spawnTask({
    type: 'local_agent',
    command: input.command,
    outputFile: generateOutputPath(),
  })
  return { taskId: task.id, status: 'started' }
}
```

任务体系带来的好处：
- 用户可以附着（attach）、终止（kill）、查看日志（logs）
- 前台 UI 不被阻塞
- 多任务可以并行，状态统一展示

## MCP Server 开发的关键点

如果你在写 MCP Server，有几件事需要注意：

**Tool schema 要精确**：MCP 工具的 inputSchema 最终会影响模型调用质量。过于宽松的 schema（如 `object: {}` ）会让模型猜参数；过于严格会让工具用起来笨重。参考内建工具的 Zod schema 写法。

**认证在 MCP Server 侧处理**：Claude Code 的 `WebFetchTool` 明确说明它不处理认证 URL。如果你要接入需要认证的外部系统，认证逻辑放在 MCP Server 里，不要依赖 Claude Code 帮你传 token。

**资源和工具分开**：MCP 协议区分 `tool`（执行动作）和 `resource`（提供数据）。读取类操作优先实现为 resource，执行类操作实现为 tool。这样权限粒度更清晰。

**测试要覆盖权限边界**：Claude Code 会对每个 MCP 工具调用做权限判断。测试时不能只测"正常调用"，要测"被拒绝时 MCP Server 怎么响应"。

## 扩展的共同约束

无论选哪种扩展方式，以下约束在 Claude Code 里都是硬性的：

1. **扩展不绕过权限层**：你的工具或 MCP server 暴露的能力，必须能被用户的 permission rule 管控。不要在工具实现里绕过 `checkPermissions`。

2. **扩展必须进消息流**：工具结果必须通过 `renderTool*Message` 进入消息系统，不要用 `console.log` 或 side effect 输出内容。

3. **扩展要处理 token 预算**：skill 加载时会被估算 token，进入上下文预算计算。如果你的 skill 很大，考虑用 `context: fork` 隔离到 subagent 执行。

4. **扩展要声明 feature flag**：如果你的能力只在特定场景开放，用 feature flag 控制注册，而不是在运行时用条件判断。

## 一张核查清单

写完一个新工具或 skill 后，过一遍这张清单：

```
□ inputSchema 有明确类型，没有 passthrough
□ checkPermissions 有真实的权限策略
□ call() 的失败路径有结构化错误，不静默吞掉
□ renderToolUseMessage 和 renderToolResultMessage 都实现了
□ renderToolUseErrorMessage 给出了可读的错误和建议
□ 长任务通过 spawnTask 接入任务体系
□ 没有在工具里维护独立 UI 状态
□ 写操作通过 deny/allow 规则可被用户管控
□ feature flag 控制了工具的可见性
□ skill 的 token 估算在合理范围内
```

# 第 8 章 Permissions 与 Approval

## 这个功能是什么

Claude Code 的权限系统负责回答一个核心问题：系统能做什么，什么时候要问用户。

Approval 是这套权限系统的用户界面。它不是提示框，而是让自动化保持可控的关键交互层——用户可以在这里给出允许、拒绝，或者写下规则让下次自动通过。

## 用户如何感知它

用户最直观的感知是：
- 某些工具调用会弹出确认（写文件、运行危险命令、访问外部 URL）
- `/permissions` 可以查看和管理已有权限规则
- 拒绝后的操作可以被重试，不会让整个任务中断

## 实现链路

权限链路通常是：

1. 工具调用前执行 `checkPermissions`
2. 工具根据文件路径、命令内容、域名等生成 permission decision
3. 如果是 `ask`，进入 approval UI，等待用户决策
4. 用户允许（可选"记住这条规则"）、拒绝，或写规则后，消息流和后续执行继续推进

规则优先级：`deny > allow > ask`。第一条命中的规则生效，后续规则不再评估。

## 关键源码点

权限系统是横切关注点，散落在各个工具里，但有一套统一协议：

```
FileEditTool   → 检查文件路径是否在 allow/deny 范围内
FileReadTool   → 检查文件读取权限
BashTool       → 检查命令级安全规则 + sandbox 策略
WebFetchTool   → 检查 hostname/domain 级白名单
AgentTool      → 检查 agent 调用和 worktree 隔离权限
```

[`commands/permissions/permissions.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/permissions.tsx) 本身很短，但信号很强：
- `/permissions` 直接返回 `PermissionRuleList`，说明权限规则是正式的产品面板，不是隐藏配置
- `onRetryDenials` 会往消息流里塞入 retry message，说明"权限拒绝"本身是会话事件，有记录

**权限规则的具体写法**：

```
# 允许所有 git 命令
allow:Bash(git *:*)

# 允许编辑 src 目录下的所有文件
allow:FileEdit(/src/**:*)

# 拒绝危险的删除操作
deny:Bash(rm -rf *:*)

# 拒绝访问生产配置
deny:FileRead(/etc/production/**:*)

# 外部域名每次都需要确认
ask:WebFetch(*.external.com:*)

# 允许特定可信域名直接通过
allow:WebFetch(api.github.com:*)
```

规则语法：`action:ToolName(matcher:*)`，其中 action 是 `allow`/`deny`/`ask`，matcher 是工具输入的匹配模式。

## 为什么这样做

如果权限系统只是全局开关，Claude Code 的自动化要么太弱（每个操作都要确认），要么太危险（完全信任模式没有任何保护）。

Claude Code 把权限判断前置到每个工具，再把 approval 做成统一界面，是为了同时满足三件事：
- **高能力**：授权后的操作无摩擦执行，不被反复打断
- **可见性**：每次确认都明确告诉用户"要做什么"
- **用户控制感**：规则可以累积，高级用户可以配置自己的自动化策略

这种设计让权限系统能随用户的使用习惯逐渐"学习"——不是通过 AI，而是通过用户写下的规则。

## 本章关键文件
- [permissions.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/permissions.tsx)
- `components/permissions/*`
- `hooks/toolPermission/*`
- `bridge/bridgePermissionCallbacks.ts`

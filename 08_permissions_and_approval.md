# 第 8 章 Permissions 与 Approval

## 这个功能是什么
Claude Code 的权限系统负责回答一个核心问题：系统能做什么，什么时候要问用户。

Approval 则是这套权限系统的用户界面。它不是提示框，而是让自动化保持可控的关键交互层。

## 用户如何感知它
用户最直观的感知是：
- 某些工具调用会弹出确认
- `/permissions` 可以查看和调整规则
- 拒绝后的操作可以被重试

## 实现链路
权限链路通常是：
1. 工具调用前先执行 `checkPermissions`
2. 工具根据文件路径、命令内容、域名等信息生成 permission decision
3. 如果是 `ask`，就进入 approval UI
4. 用户允许、拒绝或写规则后，消息流和后续执行再继续推进

## 关键源码点
[`commands/permissions/permissions.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/permissions.tsx) 本身很短，但信号很强：
- `/permissions` 直接返回 `PermissionRuleList`
- 这说明 permissions 不是隐藏设置，而是正式交互面板
- `onRetryDenials` 会往消息流里塞入 retry message，说明“权限拒绝”本身也是会话事件的一部分

再结合前面几章的工具实现可以看出：
- `FileEditTool` 做文件写权限判断
- `FileReadTool` 做读权限判断
- `WebFetchTool` 做域名级权限判断
- `BashTool` 做命令级权限和 sandbox 判断

所以 Claude Code 的权限系统不是一个总开关，而是“多工具共享的决策框架 + 一套统一确认 UI”。

## 为什么这样做
如果权限系统只是全局开关，Claude Code 的自动化要么太弱，要么太危险。

Claude Code 选择把权限判断前置到每个工具，再把 approval 做成统一界面，是因为它要同时满足：
- 高能力
- 可见性
- 用户控制感

## 本章关键文件
- [permissions.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/commands/permissions/permissions.tsx)
- `components/permissions/*`
- `hooks/toolPermission/*`
- `bridge/bridgePermissionCallbacks.ts`

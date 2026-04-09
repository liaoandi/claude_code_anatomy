# 解剖 Claude Code

Claude Code 有很多行为，用着用着会觉得理所当然，但仔细一想会发现不太对劲。这本书讲的就是这些行为背后的设计逻辑——它为什么这样做，每个决定是在什么取舍下做出的。

读者预设是已经在用 Claude Code 的人。建议先读架构总览，再按兴趣进入各章。

## 目录

**导读**
- [前言](./00_preface.md)
- [架构总览](./00b_architecture_overview.md)

**主交互功能**
- [第 1 章 对话与命令输入](./01_dialog_and_command_input.md)
- [第 2 章 消息流与会话界面](./02_message_flow_and_session_ui.md)
- [第 3 章 Slash Commands 功能族](./03_slash_commands.md)

**代码操作功能**
- [第 4 章 读文件、搜内容、找文件](./04_read_search_and_context.md)
- [第 5 章 改文件与 Diff 展示](./05_editing_and_diff.md)
- [第 6 章 Shell 执行](./06_shell_execution.md)

**任务控制功能**
- [第 7 章 Plan Mode](./07_plan_mode.md)
- [第 8 章 Permissions 与 Approval](./08_permissions_and_approval.md)
- [第 9 章 Task、Agent、Teammate](./09_tasks_agents_and_teammates.md)

**记忆与上下文功能**
- [第 10 章 Memory 功能](./10_memory.md)
- [第 11 章 Session、Resume、History](./11_session_resume_history.md)
- [第 12 章 Context 管理](./12_context_management.md)

**外部连接能力**
- [第 13 章 Web 访问与搜索](./13_web_and_external_info.md)
- [第 14 章 MCP](./14_mcp.md)
- [第 15 章 Plugins 与 Skills](./15_plugins_and_skills.md)

**运行环境功能**
- [第 16 章 Config 与 Feature Flags](./16_config_and_feature_flags.md)
- [第 17 章 Remote、Bridge、Daemon、Bg Sessions](./17_remote_bridge_daemon_bg_sessions.md)

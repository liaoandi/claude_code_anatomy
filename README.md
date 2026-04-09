# Claude Code 功能设计与实现

目标不是按目录解释每个文件，而是回答：
- Claude Code 有哪些核心功能
- 这些功能怎么设计和实现
- 为什么它会采用这种实现方式

每章固定回答五件事：这个功能是什么、用户如何感知它、实现链路、关键源码点、为什么这样做。

**建议先读架构总览，再进入各功能章节。**

## 目录

**导读**
- [前言](./src/00_preface.md)
- [架构总览](./src/00b_architecture_overview.md)

**主交互功能**
- [第 1 章 对话与命令输入](./src/01_dialog_and_command_input.md)
- [第 2 章 消息流与会话界面](./src/02_message_flow_and_session_ui.md)
- [第 3 章 Slash Commands 功能族](./src/03_slash_commands.md)

**代码操作功能**
- [第 4 章 读文件、搜内容、找文件](./src/04_read_search_and_context.md)
- [第 5 章 改文件与 Diff 展示](./src/05_editing_and_diff.md)
- [第 6 章 Shell 执行](./src/06_shell_execution.md)

**任务控制功能**
- [第 7 章 Plan Mode](./src/07_plan_mode.md)
- [第 8 章 Permissions 与 Approval](./src/08_permissions_and_approval.md)
- [第 9 章 Task、Agent、Teammate](./src/09_tasks_agents_and_teammates.md)

**记忆与上下文功能**
- [第 10 章 Memory 功能](./src/10_memory.md)
- [第 11 章 Session、Resume、History](./src/11_session_resume_history.md)
- [第 12 章 Context 管理](./src/12_context_management.md)

**外部连接能力**
- [第 13 章 Web 访问与搜索](./src/13_web_and_external_info.md)
- [第 14 章 MCP](./src/14_mcp.md)
- [第 15 章 Plugins 与 Skills](./src/15_plugins_and_skills.md)

**运行环境功能**
- [第 16 章 Config 与 Feature Flags](./src/16_config_and_feature_flags.md)
- [第 17 章 Remote、Bridge、Daemon、Bg Sessions](./src/17_remote_bridge_daemon_bg_sessions.md)

**结语**
- [从这些设计里读到什么](./src/18_conclusion.md)

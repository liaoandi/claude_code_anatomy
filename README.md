# Claude Code 功能设计与实现

这是一套基于本地源码树 `/Users/antonio/Desktop/cc2.1.88/all/src` 编写的 Claude Code 功能导读。

目标不是按目录解释每个文件，而是回答：
- Claude Code 有哪些核心功能
- 这些功能怎么设计和实现
- 为什么它会采用这种实现方式

这版已经统一升级过一轮，章节采用同一结构：
- 这个功能是什么
- 用户如何感知它
- 实现链路
- 关键源码点
- 为什么这样做

## 目录
- [前言](./00_preface.md)
- [第 1 章 对话与命令输入](./01_dialog_and_command_input.md)
- [第 2 章 消息流与会话界面](./02_message_flow_and_session_ui.md)
- [第 3 章 Slash Commands 功能族](./03_slash_commands.md)
- [第 4 章 读文件、搜文件、看上下文](./04_read_search_and_context.md)
- [第 5 章 改文件与 Diff 展示](./05_editing_and_diff.md)
- [第 6 章 Shell 执行](./06_shell_execution.md)
- [第 7 章 Plan Mode](./07_plan_mode.md)
- [第 8 章 Permissions 与 Approval](./08_permissions_and_approval.md)
- [第 9 章 Task、Agent、Teammate](./09_tasks_agents_and_teammates.md)
- [第 10 章 Session、Resume、History](./10_session_resume_history.md)
- [第 11 章 Memory 功能](./11_memory.md)
- [第 12 章 Context 管理](./12_context_management.md)
- [第 13 章 Web 与外部信息获取](./13_web_and_external_info.md)
- [第 14 章 MCP](./14_mcp.md)
- [第 15 章 Plugins 与 Skills](./15_plugins_and_skills.md)
- [第 16 章 Config 与 Feature Flags](./16_config_and_feature_flags.md)
- [第 17 章 Remote、Bridge、Daemon、Bg Sessions](./17_remote_bridge_daemon_bg_sessions.md)
- [第 18 章 状态、统计与应用壳](./18_state_stats_and_app_shell.md)

# 第 15 章 Plugins 与 Skills

## 一个共同机制，两种扩展方式

Plugins 和 skills 是 Claude Code 的高层扩展方式，用来做"工作流复用"和"自定义 slash command"，不是接入外部系统（那是 MCP 的事）。

两者的底层机制相同：写一个 markdown 文件，系统启动时扫描并编译成命令对象，并入命令注册表。用户用起来和内建命令一样，感知不到区别。

## Skill 是什么

一个 skill 就是一个 `SKILL.md` 文件，加上可选的辅助文件（脚本、模板、参考资料）。

文件头部的 frontmatter 控制这个 skill 的行为：
- `whenToUse`：告诉系统什么情况下应该用这个 skill
- `allowedTools`：限制 skill 只能调用哪些工具
- `model`、`effort`：指定用什么模型、花多大力气
- `context: fork`：在独立的 subagent 里执行，不污染主上下文

skill 加载时会被估算 token，纳入系统整体的上下文预算计算。如果 skill 文件太大，就会影响每次会话能用的上下文空间。

skill 适合封装"Claude Code 自己能完成的工作流"，比如"按这套标准审查 PR"、"用这个格式生成报告"。如果要接入 Claude Code 本身没有能力触达的外部系统（比如数据库、内部 API），那是 MCP 的事，不是 skill 能做的。

## Plugin 是什么

Plugin 是 skill 的打包形式。一个 plugin 可以包含多个 skill 和命令，安装之后统一以 plugin 名称作为命令前缀，比如 `/review:start`、`/review:summary`。

和单独的 skill 相比，plugin 多了一个能力：携带 hooks。hooks 是绑定到系统事件的自动触发脚本，不需要用户主动调用。比如每次写文件之前自动跑一次 lint 检查，或者每次会话开始时自动加载某个配置。安装 plugin 之后，这些行为就静默地生效了。

## 为什么扩展内容也要遵守系统规则

Skill 的 `allowedTools` 限制工具调用，skill 的 token 估算纳入预算，plugin 的 hooks 走统一的 hook 机制——这些不是限制扩展能力，而是让扩展内容能被系统治理。

如果 skill 可以绕过权限检查，一个写得粗糙的第三方 skill 就能在你不知情的情况下做任何事。把扩展纳入同一套规则，是扩展生态能健康运作的前提。

## 写 Skill 的几个关键点

**`whenToUse` 写清楚**。这个字段告诉系统什么情况下应该用这个 skill，写得模糊的话，系统要么永远用、要么永远不用。要具体描述触发场景，比如"当用户要审查 PR 时"，而不是"代码相关任务"。

**控制 token 体积**。Skill 文件加载进上下文，如果很大，每次会话的上下文预算都会受影响。只放必要的指导，参考资料用单独的 references 文件，不要全塞进 `SKILL.md`。

**`allowedTools` 不要留空**。不限制 allowed tools 的 skill 可以调用任何工具，包括你没想到的。明确列出这个 skill 需要的工具，既是安全考虑，也让别人一眼看清楚它的能力边界。

**考虑用 `context: fork`**。如果 skill 执行的任务比较长、会产生大量中间输出，用 fork 让它在独立的 subagent 里执行，主对话的上下文不会被污染。

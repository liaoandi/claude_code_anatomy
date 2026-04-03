# 第 15 章 Plugins 与 Skills

## 这个功能是什么

Plugins 和 Skills 是 Claude Code 的两种高层扩展方式。它们不是协议级动态工具（那是 MCP），而是面向命令复用、工作流封装和内容注入的扩展层。

核心机制是：**把 markdown 文件编译成命令对象**，并入命令注册表。

## 用户如何感知它

用户通常会感知到：
- 某些 slash command 来自插件或 skill（如 `/cleanup_project`、`/cross_agent_review_loop`）
- 某些工作流可以通过 skill 形式复用，不需要每次重写 prompt
- 不改核心代码就能扩展系统行为

## 实现链路

这套扩展的典型链路是：

1. 系统启动时扫描 plugin 目录和 skill 目录
2. 读取 markdown 文件和 frontmatter 元数据
3. 解析命令名、描述、参数、allowedTools、model 等配置
4. 把它们转换成结构化 `Command` 对象
5. 并入 `commands.ts` 的命令注册表，和内建命令统一管理

Skill 在加载时还会被估算 token，纳入系统整体上下文预算控制。

## 关键源码点

[`utils/plugins/loadPluginCommands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/utils/plugins/loadPluginCommands.ts) 说明 plugin command 本质上是"从 markdown 编译成命令对象"：
- 递归收集 markdown 文件
- skill 目录里的 `SKILL.md` 有单独处理逻辑（frontmatter 字段更丰富）
- 命令名按目录结构和 plugin name 生成 namespace（如 `plugin-name:command-name`）
- frontmatter 控制参数、描述、shell 执行方式、model 选择等行为

[`skills/loadSkillsDir.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/skills/loadSkillsDir.ts) 说明 skill 不是简单的文本片段：
- skill 有 `paths`、`hooks`、`allowedTools`、`whenToUse`、`model`、`effort`、`context` 等 frontmatter 字段
- `context: fork` 可以让 skill 在独立 subagent 里执行，不污染主上下文
- skill 按 setting source、managed path、plugin only policy 等条件筛选加载
- skill 在加载时估算 token，用于系统整体预算控制

## Plugins 与 Skills 的区别

两者都是从 markdown 加载，但定位不同：

| | Plugin | Skill |
|--|--------|-------|
| 主要用途 | 打包一组命令和工具 | 注入工作流上下文或行为指导 |
| 加载方式 | 命令扫描 + namespace | skills 目录 + `SKILL.md` 文件 |
| 执行上下文 | 命令层执行 | 可以 fork subagent 执行 |
| Token 管理 | 不估算 | 估算 token，纳入预算 |
| 触发方式 | 用户显式调用 slash command | 条件注入（`whenToUse` 字段） |

## 为什么这样做

如果扩展能力只是脚本目录，Claude Code 很快会失去统一性——每个扩展自己管权限、自己管展示、自己管上下文，维护成本很高。

把插件和技能加载做成结构化解析，收益是：
- **扩展内容仍被命令系统理解**：namespace 隔离，不和内建命令冲突
- **扩展仍被权限和上下文治理**：`allowedTools` 限制 skill 能调用哪些工具，token 估算纳入预算
- **两种扩展层次并存**：plugin 做"功能包"，skill 做"行为指导"，各司其职

## 本章关键文件
- [loadPluginCommands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/utils/plugins/loadPluginCommands.ts)
- [loadSkillsDir.ts](/Users/antonio/Desktop/cc2.1.88/all/src/skills/loadSkillsDir.ts)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)

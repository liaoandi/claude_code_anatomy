# 第 15 章 Plugins 与 Skills

## 这个功能是什么
Plugins 和 Skills 是 Claude Code 的两种高层扩展方式。它们不是协议级动态工具，而是面向命令、工作流和内容复用的扩展层。

## 用户如何感知它
用户通常会感知到：
- 某些命令来自插件或 skill
- 某些工作流可以通过 markdown 或 skill 形式复用
- 系统可以不改核心代码就扩展行为边界

## 实现链路
这套扩展的一条典型链路是：
1. 系统扫描 plugin 或 skill 目录
2. 读取 markdown/frontmatter
3. 解析命令名、描述、参数、可用工具、model/frontmatter 配置
4. 把它们转换成正式 `Command` 并挂回命令系统

## 关键源码点
[`utils/plugins/loadPluginCommands.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/utils/plugins/loadPluginCommands.ts) 说明 plugin command 本质上是“从 markdown 编译成命令对象”：
- 会递归收集 markdown 文件
- skill 目录里的 `SKILL.md` 有单独处理逻辑
- 命令名按目录结构和 plugin name 生成 namespace
- frontmatter 会控制参数、描述、shell、model 等行为

[`skills/loadSkillsDir.ts`](/Users/antonio/Desktop/cc2.1.88/all/src/skills/loadSkillsDir.ts) 则说明 skill 不是简单文本片段：
- skill 有 `paths`、`hooks`、`allowedTools`、`whenToUse`、`model`、`effort` 等 frontmatter 字段
- skill 可以按 setting source、managed path、plugin only policy 等条件加载
- skill 会被估算 token，用于系统整体预算控制

所以 Plugins 与 Skills 的真正价值不是“多一些命令”，而是“把扩展也纳入同一套结构化系统”。

## 为什么这样做
如果扩展能力只是脚本目录，Claude Code 很快会失去统一性。
它把插件和技能加载做成结构化解析，就是为了：
- 扩展内容仍然能被命令系统理解
- 扩展仍然能被权限、上下文、token 预算治理
- 插件级扩展和工作流级扩展可以并存

## 本章关键文件
- [loadPluginCommands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/utils/plugins/loadPluginCommands.ts)
- [loadSkillsDir.ts](/Users/antonio/Desktop/cc2.1.88/all/src/skills/loadSkillsDir.ts)
- [commands.ts](/Users/antonio/Desktop/cc2.1.88/all/src/commands.ts)

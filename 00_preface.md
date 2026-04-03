# 前言

## 这本书讲什么

这本书基于本地源码树 `/Users/antonio/Desktop/cc2.1.88/all/src`，是一份 Claude Code 的功能导读。

它不按源码目录机械讲解，而是按功能域回答三个问题：
- Claude Code 到底提供了什么能力
- 这些能力在源码里是怎么落地的
- 为什么实现会长成这个样子

本书默认你已经是 Claude Code 的重度用户，不解释基本用法，也不重复官方文档里的入门内容。

## 这本书不讲什么

本书不做三类事情：
- 不讲"如何安装和使用 Claude Code"
- 不把 OpenClaw 或其他 agent 系统混入对比
- 不沉迷于过细的 UI 样式或零碎 helper

你真正要学的是功能设计和实现抓手，不是把每个 utils 文件都过一遍。

## 如何阅读

建议采用固定顺序：
1. 先读入口和命令注册
2. 再读对应功能的 command / tool / service
3. 最后回到 UI 组件和状态容器

原因很简单：Claude Code 的很多功能不是从组件长出来的，而是先有运行时意图，再由 UI 承接。

一个实用的阅读套路：
- 先找命令入口或工具定义
- 再看 `call`、`checkPermissions`、`inputSchema`、`renderTool*Message`
- 再看它依赖的 service 或 utils
- 最后看界面如何把它显示出来

## 全书功能地图

整本书分成六段：

| 段落 | 章节 | 内容 |
|------|------|------|
| 主交互功能 | 1-3 | 输入、消息流、slash commands |
| 代码操作功能 | 4-6 | 读文件、改文件、shell |
| 任务控制功能 | 7-9 | plan、permissions、task/agent |
| 记忆与上下文功能 | 10-12 | memory、session、context |
| 外部连接能力 | 13-15 | web、MCP、plugins/skills |
| 运行环境功能 | 16-18 | config、remote/bridge/daemon、app shell |

建议先读「架构总览」章节，再进入各功能章节。

每章固定回答五件事：这个功能是什么、用户如何感知它、实现链路是什么、关键源码点在哪里、为什么这样做。

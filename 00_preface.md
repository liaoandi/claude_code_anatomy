# 前言

## 这本书讲什么
这本书是基于本地源码树 `/Users/antonio/Desktop/cc2.1.88/all/src` 写的 Claude Code 功能导读。

它不按源码目录做机械讲解，而是按功能域回答三个问题：
- Claude Code 到底提供了什么能力
- 这些能力在源码里是怎么落下来的
- 为什么实现会长成这个样子

这本书默认你已经是 Claude Code 重度用户，所以不解释基本用法，也不重复官方文档里的入门内容。

## 这本书不讲什么
这本书不做三类事情：
- 不讲“如何安装和使用 Claude Code”
- 不把 OpenClaw 或别的 agent 系统混进来对比
- 不沉迷于过细的 UI 样式或零碎 helper

你真正要学的是功能设计和实现抓手，不是把每个 utils 文件都过一遍。

## 如何阅读这份源码
建议采用固定顺序：
1. 先看入口和命令注册
2. 再看对应功能的 command / tool / service
3. 最后再回到 UI 组件和状态容器

原因很简单：Claude Code 的很多功能不是从组件长出来的，而是先有运行时意图，再由 UI 去承接。

一个实用阅读套路是：
- 先看命令入口或工具定义
- 再看 `call`、`checkPermissions`、`inputSchema`、`renderTool*Message`
- 再看它依赖的 service 或 utils
- 最后再看界面如何把它显示出来

## Claude Code 的总功能地图
整本书分成六段：
- 主交互功能：输入、消息流、slash commands
- 代码操作功能：读文件、改文件、shell
- 任务控制功能：plan、permissions、task/agent
- 记忆与上下文功能：session、memory、context
- 外部连接能力：web、MCP、plugins/skills
- 运行环境功能：config、remote/bridge/daemon、app shell

每章都会固定回答五件事：
- 这个功能是什么
- 用户如何感知它
- 实现链路是什么
- 关键源码点在哪里
- 为什么这样做

# 架构总览

在进入各功能章节之前，先把整个系统的骨架看清楚。这一章不讲细节，只讲"各层是什么、相互关系是什么、为什么这样分"。

## 六个层次，从外到内

Claude Code 的架构可以分成六个层次，从用户可见的最外层到系统内部最底层：

```
┌─────────────────────────────────────────────────┐
│               入口层 Entry Layer                 │
│   cli.tsx：模式分流、fast path、动态加载          │
├─────────────────────────────────────────────────┤
│              命令层 Command Layer                │
│   commands.ts：slash command 注册表              │
├─────────────────────────────────────────────────┤
│               工具层 Tool Layer                  │
│   FileRead / FileEdit / Bash / Web / Agent / MCP │
├──────────────────────┬──────────────────────────┤
│   权限层 Permission  │   消息层 Message Layer    │
│   checkPermissions   │   renderTool* / Messages  │
│   每个工具的前置判断  │   统一展示协议            │
├──────────────────────┴──────────────────────────┤
│              任务层 Task Layer                   │
│   Task.ts：long-running 执行的统一抽象           │
├─────────────────────────────────────────────────┤
│         状态与上下文层 State / Context Layer      │
│   Memory / Session / Context / Config / Feature  │
└─────────────────────────────────────────────────┘
```

## 各层职责

**入口层**：决定当前是什么运行模式。CLI 启动后先分流——是 `daemon`、`bridge`，还是普通交互 REPL，还是 `ps/logs/attach/kill` 这类管理命令。只有普通交互路径才继续加载完整主程序。这一层还承担 fast path（如 `--version` 零模块加载）和动态 `import()` 懒加载，压冷启动成本。

**命令层**：负责"哪些 slash command 存在、可见、可调用"。`commands.ts` 是产品能力的完整目录，命令通过 feature flag 条件注册，插件和 skill 也会把自己的命令并入这里。命令不是散乱函数，而是结构化对象，有类型、描述、参数和执行逻辑。

**工具层**：执行实际能力的地方。每个工具都是标准结构体：`inputSchema`（定义接受什么）、`checkPermissions`（是否允许执行）、`call`（执行逻辑）、`renderToolUseMessage` / `renderToolResultMessage`（如何在界面展示）。工具之间独立，通过任务层和权限层共享基础设施。

**权限层**：横切所有工具的前置判断，不是总开关，而是"每个工具在每次调用时按具体上下文决策"。文件写操作检查路径权限，Shell 命令检查命令级安全规则，Web 请求检查域名白名单，Agent 调用有自己的隔离策略。拒绝后的 approval UI 统一由权限层提供，不由各工具自己实现。

**消息层**：把所有工具输出、状态变化、权限拒绝、diff 结果统一纳入消息流。工具不是"返回数据让 UI 猜怎么画"，而是天然带显示方式——`renderTool*Message` 是工具协议的一部分。`Messages.tsx` 再做 normalize、reorder、grouping、collapse，最终呈现为连贯的会话轨迹。

**任务层**：长生命周期执行的统一抽象。Shell 命令、subagent、remote job、teammate 都被注册为 Task，有统一的 `TaskType`、`TaskStatus`、`outputFile`，生命周期从 `pending → running → completed/failed/killed`。这让前台 UI 和后台执行解耦，多执行者统一呈现。

**状态与上下文层**：系统的记忆和配置基础。包括 Memory（长期记忆提取）、Session/History（会话续接）、Context（当前窗口预算控制）、Config（运行参数）、Feature Flag（功能开关）。这一层不承载业务逻辑，只为其他层提供持久状态。

## 三个贯穿全系统的设计原则

**能力产品化，不留给 shell 临时解决**
读文件有 FileReadTool，改文件有 FileEditTool，搜索有 GrepTool 和 GlobTool，联网有 WebFetchTool。这些能力被做成一级工具，而不是让模型自己组合 shell 命令临时解决。产品化带来的收益是：每种能力都有权限检查、展示协议、审计轨迹，而不是黑盒执行。

**展示即协议，工具自带显示方式**
每个工具不只定义"做什么"，还定义"怎么显示"。`renderToolUseMessage` 和 `renderToolResultMessage` 是工具接口的正式部分。这让会话可回放、状态可审计，也让所有工具复用同一套展示容器，不用各自发明 UI。

**扩展不破坏内聚**
MCP 通过协议在运行时动态注入工具，插件和 skill 通过 markdown 编译成命令对象并入命令层，feature flag 控制哪些能力在哪个产品版本暴露。扩展机制的共同特点是：外部能力仍然必须遵守权限层、消息层和任务层的规则，不存在"绕过系统"的后门。

## 如何利用这张地图读后续章节

后续 18 章，每章讲一个功能域。读的时候可以对照这张地图定位：

- 第 1-3 章：入口层 + 命令层的细节
- 第 4-6 章：工具层的三个核心工具（Read / Edit / Bash）
- 第 7-9 章：权限层（Plan Mode 是权限上下文切换）+ 任务层
- 第 10-12 章：状态与上下文层（Memory / Session / Context）
- 第 13-15 章：扩展能力（Web / MCP / Plugins & Skills）
- 第 16-18 章：运行环境（Config / Feature Flag / Remote / App Shell）

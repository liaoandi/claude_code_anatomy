# 第 18 章 状态、统计与应用壳

## 这个功能是什么
Claude Code 顶层的 `App` 很薄，但它承担的是“把全局能力兜起来”的职责：状态、统计、性能指标和全局上下文都从这里灌进组件树。

这一章只讲顶层壳，不重复展开各功能域内部的状态机。

## 用户如何感知它
用户通常不会直接注意到 `App.tsx`，但会感知到几个结果：
- 界面状态是连续的
- 统计和性能信息能稳定存在
- 新功能继续加进去时，顶层并没有变得混乱

## 实现链路
App 壳的链路非常简单：
1. 接收 `initialState`、`stats`、`getFpsMetrics`
2. 依次套上 `AppStateProvider`、`StatsProvider`、`FpsMetricsProvider`
3. 通过 `onChangeAppState` 统一监听状态变化

## 关键源码点
[`components/App.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/App.tsx) 很薄，几乎只做 provider 组合：
- `AppStateProvider`
- `StatsProvider`
- `FpsMetricsProvider`

这说明 Claude Code 很克制地把顶层当成“壳”，而不是“业务总线”。

这类设计的价值在于：
- 业务逻辑下沉到 commands / tools / services / hooks
- 顶层只承接全局环境和观测能力
- 状态变化可以集中追踪，但不迫使所有功能都堆到一层

## 为什么这样做
功能越来越多时，最容易膨胀的就是顶层壳。Claude Code 这里反而保持了很强的纪律性：
- 壳薄
- provider 明确
- 业务下沉

这让系统可以继续长，而不用不断重写最外层架构。

## 本章关键文件
- [App.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/App.tsx)
- `state/*`
- `context/*`

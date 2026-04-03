# 第 18 章 状态、统计与应用壳

## 这个功能是什么

Claude Code 顶层的 `App` 很薄，但承担的是"把全局能力兜起来"的职责：状态、统计、性能指标和全局上下文都从这里灌进组件树。

这一章只讲顶层壳，不重复展开各功能域内部的状态机。

## 用户如何感知它

用户通常不会直接注意到 `App.tsx`，但会感知到几个结果：
- 界面状态是连续的，不同功能之间切换不会丢失上下文
- 统计和性能信息能稳定存在，`/stats` 之类的命令始终可用
- 功能越加越多，但顶层体验没有明显变乱

这些都是顶层壳做好了才有的结果，但用户感知不到壳本身。

## 实现链路

App 壳的链路非常简单：

1. 接收 `initialState`、`stats`、`getFpsMetrics` 等全局入参
2. 依次套上 `AppStateProvider`、`StatsProvider`、`FpsMetricsProvider`
3. 通过 `onChangeAppState` 统一监听状态变化
4. 其他所有功能从 provider 消费这些全局能力，不直接依赖 App

## 关键源码点

[`components/App.tsx`](/Users/antonio/Desktop/cc2.1.88/all/src/components/App.tsx) 几乎只做 provider 组合，没有业务逻辑：

```
AppStateProvider        // 全局应用状态
  StatsProvider         // token 统计、用量数据
    FpsMetricsProvider  // 渲染性能指标
      <子组件树>
```

这说明 Claude Code 很克制地把顶层当成"壳"，而不是"业务总线"：
- 业务逻辑下沉到 commands / tools / services / hooks
- 顶层只承接全局环境和观测能力
- 状态变化可以集中追踪，但不迫使所有功能都堆到一层

`state/*` 和 `context/*` 目录维护各自功能域的状态，App 只是把它们的 provider 组合起来，不直接管理业务状态。

## 为什么这样做

功能越来越多时，最容易膨胀的就是顶层壳。很多系统最后变成了"大 App.tsx"——什么状态都放在顶层，什么副作用都从顶层触发，改任何功能都要动顶层。

Claude Code 这里反而保持了很强的纪律性：
- **壳薄**：App.tsx 只做 provider 组合，代码量极少
- **provider 明确**：每类全局能力对应一个 provider，边界清晰
- **业务下沉**：功能逻辑不在顶层，在各自的 commands/tools/services 里

这让系统可以继续长，而不用不断重写最外层架构——顶层稳定，功能在下层迭代。

## 本章关键文件
- [App.tsx](/Users/antonio/Desktop/cc2.1.88/all/src/components/App.tsx)
- `state/*`
- `context/*`

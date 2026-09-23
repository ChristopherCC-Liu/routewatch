# RouteWatch

RouteWatch 是一个只读终端面板，用于汇总 Codex、Claude Code 和 Hermes 在本机记录的模型路由与 token 用量。

## 启动

```sh
routewatch
```

按键：`1` 查看 1 天，`2` 查看 7 天，`3` 查看 30 天，`4` 查看 90 天，`r` 刷新文件列表，`q` 退出。默认窗口为 30 天。

查看一次文本快照：

```sh
routewatch --once
```

运行 `routewatch --help` 可更改来源目录、Hermes 数据库路径或刷新间隔。

## 数据来源与口径

- **Codex：**读取 `~/.codex/sessions/**/*.jsonl` 的逐次用量事件，并从会话上下文读取模型和 provider。
- **Claude Code：**读取 `~/.claude/projects/**/*.jsonl` 的 assistant 用量记录；同一条流式消息的重复记录会合并为最完整的一条。本地用量记录有模型名，但不含所用传输 provider，因此面板会显示“provider unknown”。
- **Hermes：**读取 `~/.hermes/state.db` 的模型用量汇总，并从 `~/.hermes/gateway_state.json` 获取 gateway 状态。
- 程序只读用量元数据，不读取 prompt、回复内容、工具参数、API key 或 provider URL；不会联网，也不会改运行配置。
- Codex 和 Claude Code 的日趋势来自带时间戳的逐次记录。Hermes 按 session/model 保存累计用量，因此图表把整个 session 计入最后活跃日；跨来源逐日比较时需留意这个近似口径。
- Prompt 数包含来源单独记录的缓存 token。Cache 栏是其中的缓存部分，不会再次加进总量。
- 速率面板统计最近一分钟写入的已完成请求记录。本机数据源不提供生成过程中的实时流式 TPS。Hermes 速率从 RouteWatch 启动并观察到数据库增量后开始累计。

只依赖 Python 3 标准库，无需安装额外软件包。

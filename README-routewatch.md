# RouteWatch

RouteWatch 是一个只读的终端面板，汇总本机 Codex、Claude Code 和 Hermes 的模型路由及 token 用量。它使用 Python 标准库，不需要安装 Python 第三方依赖，不连接网络，也不修改这些工具的配置。

## 安装

### macOS

确保已安装 Python 3.10 或更新版本，并能在终端运行 `python3 --version`。macOS 自带的 Python 版本过旧时，可先安装 Homebrew，再运行 `brew install python`。

```sh
git clone https://github.com/ChristopherCC-Liu/routewatch.git
cd routewatch
mkdir -p "$HOME/.local/bin"
install -m 755 routewatch "$HOME/.local/bin/routewatch"
```

确保 `~/.local/bin` 在 PATH 中。zsh 用户可以执行下面这段，将它加入 `~/.zshrc`（若配置中还没有）：

```sh
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

重新打开终端后，输入 `routewatch` 即可启动。也可以不安装到 PATH，直接在仓库目录执行 `./routewatch`。

### Linux

安装 Python 3.10+、Git 和系统 curses 支持。Debian/Ubuntu 可运行：

```sh
sudo apt install python3 python3-venv git
git clone https://github.com/ChristopherCC-Liu/routewatch.git
cd routewatch
mkdir -p "$HOME/.local/bin"
install -m 755 routewatch "$HOME/.local/bin/routewatch"
```

确认 `~/.local/bin` 在 `PATH` 中，然后输入 `routewatch` 启动。如果 Python 构建未包含 curses，请安装发行版提供的 Python curses/ncurses 支持。

### 更新

```sh
cd routewatch
git pull
install -m 755 routewatch "$HOME/.local/bin/routewatch"
```

如果安装时用了不同目录，请将最后一行的目标目录改成实际位置。

## 启动与选项

```sh
routewatch                 # 启动交互终端面板，默认回看30天
routewatch --days 1        # 今天：本地零点到现在，按小时统计
routewatch --days 7        # 最近7天，按天统计
routewatch --days 90       # 最近90天，按天统计
routewatch --refresh 1     # 每秒刷新面板
routewatch --once          # 输出一次文本快照后退出
routewatch --help          # 查看所有参数
```

第一次启动会先立即显示界面，历史日志会在后台索引。索引期间可照常切换视图和退出。窗口建议至少 80×24；终端支持鼠标时也可点击页签、来源、趋势柱和路由行。

## 交互操作

| 操作 | 按键 |
|---|---|
| 切换总览、路由、趋势、来源视图 | `Tab` / `Shift-Tab`，也可点击页签 |
| 时间范围 | `1` 今天、`2` 7天、`3` 30天、`4` 90天 |
| 来源筛选 | `f` 循环切换 All / Codex / Claude Code / Hermes |
| 路由列表导航 | `↑` / `↓` 或 `j` / `k`，`PgUp` / `PgDn`，`Home` / `End` |
| 查看路由详情 | `Enter` 打开；`Esc` 或 `Enter` 关闭；鼠标双击也可打开 |
| 搜索模型或 backend | 在路由页按 `/`，输入关键词后按 `Enter` |
| 排序 / 清除筛选 | `s` 切换排序；`x` 清空搜索和来源筛选 |
| 检查趋势数据 | 左右方向键或 `h` / `l` 移动游标，`Home` / `End` 跳转 |
| 趋势指标 | `m` 在总量、prompt、输出、缓存、调用次数之间切换 |
| 暂停 / 恢复显示 | 空格；采集在后台继续，切换来源或时间范围会恢复显示 |
| 刷新 / 帮助 / 退出 | `r` / `?` / `q` |

总览页展示 token 总量、调用次数、缓存占比、速率、来源占比和主要路由。来源占比行可点击进入该来源。路由详情显示模型、来源、backend/mode/task、调用数、各类 token 和最近记录时间。来源页展示三种采集源状态、日志/用量记录数、最新用量时间和来源速率。

## 数据来源和指标口径

- **Codex：**读取 `~/.codex/sessions/**/*.jsonl` 的逐次用量事件，从会话上下文提取模型和 provider。
- **Claude Code：**读取 `~/.claude/projects/**/*.jsonl` 中的 assistant 用量记录；流式输出可能重复写入同一消息，程序会合并并采用用量最完整的记录。日志没有传输 provider 字段，所以显示 `provider unknown`。
- **Hermes：**只读 `~/.hermes/state.db` 中的模型用量汇总，并读取 `~/.hermes/gateway_state.json` 显示 gateway 状态。Hermes 按 session/model 保存累计用量，历史图表把该 session 的总量归在最后活跃日，这是近似口径。
- 总量是 prompt 与输出之和。Prompt 包含各来源记录的缓存 token；缓存是 prompt 的子集，只用于单独展示，不会重复计入总量。
- 面板的 `/s` 速率按最近一分钟内落盘的已完成用量记录估算。Codex 和 Claude Code 不把当前生成过程中的流式 token 暴露在本地日志中，因此它不是实时生成 TPS。Hermes 速率从 RouteWatch 启动并观测到数据库数值变化后才开始计算。
- 来源筛选同步应用于总览、路由和趋势；来源诊断页始终保留全部来源。搜索只过滤路由列表。
- 如果某个工具使用自定义数据目录，可用 `--codex-dir`、`--claude-dir`、`--hermes-db`、`--hermes-state` 指定位置。

## 故障排查

- `routewatch: command not found`：确认安装目标 `~/.local/bin/routewatch` 存在，且 `~/.local/bin` 已在 `PATH` 中；运行 `echo "$PATH"` 检查，或直接使用 `./routewatch`。
- 没有路由记录：确认对应工具在本机已有会话记录，检查来源页的连接状态，并确认传给参数的目录或数据库位置正确。
- `curses` 或终端错误：先用兼容的终端模拟器，确认 `TERM` 有设置；如果是通过管道或非交互 shell 运行，使用 `routewatch --once` 获取文本输出。
- 数据较多时首轮历史索引可能需要一些时间；面板会先打开，索引完成后自动更新。

RouteWatch 只读取并汇总本机日志和数据库记录。实现依赖 Python 3 标准库，没有第三方运行时依赖。

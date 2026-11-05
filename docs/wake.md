# Wake

- 仓库地址：https://github.com/iAmCorey/Wake
- 官网：https://github.com/iAmCorey/Wake

Agent 会话管理器（基于 GPUI / Rust）：浏览、搜索、恢复各家 coding agent 的会话。桌面端为 `wake` GUI，另有 `wake-cli` 命令行和 `wake-mcp`（给 agent 用的 MCP 服务端）。

会话数据在 `~/.local/share/wake/wake.db`，同一个数据库只能被一个进程打开，运行中的 `wake` 会让其它 `wake` 子命令报「另一个进程正在使用它」。

## 资产与安装（按平台）

### Linux

| 架构 | tar.gz | deb |
| --- | --- | --- |
| x86_64 | `wake-<版本>-linux-amd64.tar.gz` | `wake_<版本>_amd64.deb` |
| aarch64 | `wake-<版本>-linux-arm64.tar.gz` | `wake_<版本>_arm64.deb` |

tar.gz 里带 `install.sh`，装到用户目录，不需要 root：

```bash
tar -xzf wake-<版本>-linux-amd64.tar.gz
cd wake-<版本>-linux-amd64
bash install.sh
```

它做三件事：`wake` / `wake-cli` / `wake-mcp` 装到 `~/.local/bin/`；`.desktop` 装到 `~/.local/share/applications/wake.desktop`，并把 `Exec` 改成 `~/.local/bin/wake` 绝对路径（否则图形会话的 PATH 里没有 `~/.local/bin`，菜单点了会找不到程序）；图标装到 `~/.local/share/icons/hicolor/scalable/apps/wake.svg`。

deb 走系统包管理器，需要 sudo：

```bash
sudo apt install ./wake_<版本>_amd64.deb
```

deb 装到 `/usr/bin`，`Exec=wake` 裸名就能用，不受上面那个 PATH 问题影响。

### macOS

资产：`Wake-<版本>-macos.zip`。解压得到 `Wake.app`，拖进 `/Applications`（或 `~/Applications`）。图形界面通过 `.app` 启动；`wake-cli` / `wake-mcp` 需要单独放进 `~/.local/bin/`。

### Windows

资产：`wake-<版本>-windows-x86_64.zip`。解压后运行 `wake.exe`；`wake-cli.exe` / `wake-mcp.exe` 可放到 PATH 目录里。

## 注意

- `wake` 是 GUI 程序，命令行下执行 `wake --version` 会尝试打开数据库并阻塞；查版本用 `wake-cli --version`。
- 升级前先退出正在运行的 `wake`，否则会锁住数据库。

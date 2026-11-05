# app-downloader

按应用名从 GitHub Release 下载应用并安装到本地。

## 组成

- `docs/<应用名>.md`：每个应用一个文件，记录仓库地址、各平台资产命名规则、安装方式。
- `.claude/skills/download-app/SKILL.md`：下载 skill，告诉 Claude 如何查 release、挑资产、校验并安装。

## 用法

在 Claude Code 里直接说：

- “下载 Clash Verge Rev”
- “装一下 CC Switch”

Claude 会先在 `docs/` 里找对应应用；没有就搜索官方仓库并补一份文档，然后按当前平台挑选 release 资产、校验并安装。

目前已有文档：

| 应用 | 仓库 |
| --- | --- |
| [Clash Verge Rev](docs/clash-verge.md) | clash-verge-rev/clash-verge-rev |
| [CC Switch](docs/cc-switch.md) | farion1231/cc-switch |
| [Zed](docs/zed.md) | zed-industries/zed |
| [Wake](docs/wake.md) | iAmCorey/Wake |
| [网易云音乐 GPUI](docs/netease-music-gpui.md) | bougieL/netease-music-gpui |
| [ChatAgent](docs/chat-agent.md) | bougieL/chat-agent-releases |
| [Tessera](docs/tessera.md) | bougieL/terminal-manager-releases |

## 约定

- 临时文件放在项目 `tmp/` 目录。
- 文档按平台分节写清楚资产命名和安装方式，不写死某一平台的路径。
- 安装位置按平台区分：Linux 命令行工具装 `~/.local/bin`，系统级安装用 deb/rpm；macOS 用 `~/.local/bin` 或 `/Applications`；Windows 走安装器或加进 PATH。
- 仓库与文档中不出现任何私有地址或内网信息。

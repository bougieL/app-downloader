# app-downloader

按应用名从 GitHub Release 下载应用并安装到本地；同时收录一批通过包管理器安装的命令行工具（CLI）。

## 组成

- `docs/<应用名>.md`：每个应用一个文件，记录仓库地址、各平台资产命名规则、安装方式。
- `docs/tv/`、`docs/games/`：按使用场景分类的文档，各文件仍按应用划分。
- `.claude/skills/download-app/SKILL.md`：下载 skill，告诉 Claude 如何查 release、挑资产、校验并安装；
  CLI 工具走同一套文档，额外强调更新时先检测原安装方式。

## 用法

在 Claude Code 里直接说：

- “下载 Clash Verge Rev”
- “装一下 CC Switch”
- “更新 opencode”

Claude 会先在 `docs/` 里找对应应用；没有就搜索官方仓库并补一份文档，然后按当前平台挑选 release 资产、校验并安装。

目前已有文档：

| 应用 | 仓库 | 类型 |
| --- | --- | --- |
| [Clash Verge Rev](docs/clash-verge.md) | clash-verge-rev/clash-verge-rev | 桌面客户端 |
| [CC Switch](docs/cc-switch.md) | farion1231/cc-switch | 桌面客户端 |
| [Zed](docs/zed.md) | zed-industries/zed | 桌面客户端 |
| [Wake](docs/wake.md) | iAmCorey/Wake | 桌面客户端 |
| [网易云音乐 GPUI](docs/netease-music-gpui.md) | bougieL/netease-music-gpui | 桌面客户端 |
| [ChatAgent](docs/chat-agent.md) | bougieL/chat-agent-releases | 桌面客户端 |
| [Tessera](docs/tessera.md) | bougieL/terminal-manager-releases | 桌面客户端 |
| [Herdr CLI](docs/herdr.md) | herdrdev/herdr | CLI |
| [OpenCode](docs/opencode.md) | anomalyco/opencode | CLI |
| [Pi](docs/pi.md) | badlogic/pi-mono / earendil-works/pi | CLI |
| [Codex CLI](docs/codex.md) | openai/codex | CLI |
| [Claude Code](docs/claude.md) | anthropics/claude-code | CLI |
| [MMX CLI](docs/mmx-cli.md) | MiniMax-AI/cli | CLI |
| [pnpm](docs/pnpm.md) | pnpm/pnpm | CLI（包管理器） |
| [微信 Linux 版](docs/wechat.md) | 官方站点分发（非 GitHub Release） | 桌面客户端 |
| [飞书 Feishu](docs/feishu.md) | 官方站点分发（非 GitHub Release） | 桌面客户端 |
| [CodeBuddy Code](docs/codebuddy.md) | codebuddy/codebuddy-code | CLI |
| [Oh My Zsh](docs/oh-my-zsh.md) | ohmyzsh/ohmyzsh | Shell 框架 |
| [PhotoCraft](docs/photocraft.md) | storytold/photocraft | 桌面客户端 |
| [Ghostty](docs/ghostty.md) | ghostty-org/ghostty | 桌面客户端 |
| [Lichii](docs/pc-games/lichii.md) | zortos293/Lichii | 模拟器 |
| [TotK Recomp](docs/pc-games/totk-recomp.md) | valentderah/totk-pc-recomp | 游戏重编译 |
| [Savage Wind Recomp](docs/pc-games/botw-recomp.md) | Revivedx/SavageWindRecomp | 游戏重编译 |
| [Longshot](docs/pc-games/longshot.md) | MagicBeanDev/Longshot（Forgejo） | 游戏重编译 |
| [VS Code](docs/vscode.md) | microsoft/vscode | 桌面客户端 |
| [Convey](docs/convey.md) | gitlab.gnome.org/donnybeelo/convey | 桌面客户端（邮件） |
| [GNOME 扩展](docs/gnome-extensions.md) | extensions.gnome.org（非 GitHub Release） | 桌面增强 |

非应用类的文档：

- [系统字体](docs/fonts.md)：全机统一的等宽/中文字体方案（Maple Mono Normal CN）、
  各应用的字体设置与 fontconfig 配置。

按场景分类的文档：

- `docs/tv/`：电视端应用（Aerial TV、Moonlight、SmartTube 等）。
- `docs/games/`：游戏相关（[Steam](docs/games/steam.md) 的音频链路与爆音修复等）。
- `docs/pc-games/`：PC 上的主机游戏模拟器与游戏移植（[Lichii](docs/pc-games/lichii.md) 的 Release 资产规则与安装、[TotK Recomp](docs/pc-games/totk-recomp.md) 的 Setup 安装器与更新、[Savage Wind Recomp](docs/pc-games/botw-recomp.md) 与 [Longshot](docs/pc-games/longshot.md) 的静态重编译版）。

个别项目不走 GitHub Release：Herdr CLI 同时提供安装脚本、brew、mise 等方式，
细节见各自文档。VS Code 的 GitHub release 里也**没有任何二进制资产**，
二进制由微软自己的更新服务器 `update.code.visualstudio.com` 和 apt / rpm 仓库分发，
详见 [docs/vscode.md](docs/vscode.md)。

## 国内镜像

把 dnf、flatpak、npm / pnpm、pip、cargo、go 等源指向国内镜像的配置文件、验证方法和踩过的坑
（重点：dnf 的 metalink 优先级高于 baseurl，只改 baseurl 是不生效的），
见 [docs/mirrors.md](docs/mirrors.md)。

## CLI 工具的安装与更新

CLI 工具（opencode、pi、codex、claude 等）不走 GitHub Release 下载，各自的安装方式不同：
官方安装脚本、npm / pnpm / bun / yarn、brew、winget、系统包管理器……

每份 CLI 文档都包含三节：

- **安装方式清单**：每种方式怎么装、怎么认出来；
- **检测当前安装方式**：用真实路径的片段 + 各包管理器的全局列表判断当初是怎么装的；
- **更新**：沿用检测出来的那种方式更新。

**更新时必须沿用原来的安装方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。判断不出来就问用户，不要猜。

## zsh 支持

本项目装的 CLI 工具都要能在 zsh 下直接用、最好还能按 Tab 补全。
统一由 `~/.oh-my-zsh/custom/conf/project-cli.zsh` 管理：

- **PATH**：各工具的安装目录（`~/.local/bin`、`~/.opencode/bin`、mise 的 shims、
  pnpm 全局目录等）都加进 `path`；
- **补全**：工具提供了 `completion` 子命令的，把补全写到
  `~/.local/share/zsh/site-functions/_<命令>`，该目录在 `fpath` 里，由 `compinit` 自动注册；
  补全文件比可执行文件旧时自动重新生成。

每个 CLI 文档末尾都有一节「zsh 支持」，写明这个工具的补全从哪来（或者没有）。
详见 [docs/oh-my-zsh.md](docs/oh-my-zsh.md)。

## 约定

- 临时文件放在项目 `tmp/` 目录。
- 文档按平台分节写清楚资产命名和安装方式，不写死某一平台的路径。
- 不写死本机绝对路径：用 `~` 开头，或用占位符（如 `<codex 家目录>`）。
- 判定安装方式用路径片段而不是完整目录，默认位置常被 `CODEX_HOME`、`PI_CODING_AGENT_DIR`
  这类环境变量改掉。
- 安装位置按平台区分：Linux 命令行工具装 `~/.local/bin`，系统级安装用 deb/rpm；macOS 用 `~/.local/bin` 或 `/Applications`；Windows 走安装器或加进 PATH。
- 仓库与文档中不出现任何私有地址或内网信息。

## 许可证

[MIT](LICENSE)

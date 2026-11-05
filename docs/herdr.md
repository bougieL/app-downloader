# Herdr CLI

- 命令：`herdr`
- 仓库地址：https://github.com/herdrdev/herdr
- 官网：https://herdr.dev
- 文档：https://herdr.dev/docs/
- Release：https://github.com/herdrdev/herdr/releases

面向 coding agent 的终端工作区管理器（Rust 单二进制，无 Electron）。把终端组织成
workspace / tab / pane，识别 pane 里跑的 agent，并通过 `herdr` CLI 和 socket API 暴露出来。
桌面客户端是另一个东西，见 [Herdr App](herdr-app.md)。

## 二进制与版本清单

GitHub Release 上的资产是**没有扩展名的裸二进制**，按 `<产品>-<平台>-<架构>` 命名：

| 平台 | 资产名 |
| --- | --- |
| Linux x86_64 | `herdr-linux-x86_64` |
| Linux aarch64 | `herdr-linux-aarch64` |
| macOS x86_64 | `herdr-macos-x86_64` |
| macOS aarch64 | `herdr-macos-aarch64` |
| Windows x86_64 | `herdr-windows-x86_64.zip` |

同一份 release 里还有 preview 通道的 tag（形如 `preview-2026-10-14-<sha>`），正式更新只看
`v*` 的正式版。

带校验和的清单在 https://herdr.dev/latest.json ，结构是扁平的：

```json
{
  "version": "0.9.5",
  "assets": { "linux-x86_64": "https://github.com/herdrdev/herdr/releases/download/v0.9.5/herdr-linux-x86_64" },
  "sha256": { "linux-x86_64": "14d9e8bc..." }
}
```

官方安装脚本和 `herdr update` 都用这份清单，**两者是一致的**，所以按脚本装的更新时
也可以直接用 `herdr update`。

## 安装方式清单

**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方安装脚本 | 真实路径是 `~/.local/bin/herdr` 这样的**普通文件**（不是软链），且不在包管理器目录里 | `herdr update` |
| 手动下载 Release 二进制 | 同上（脚本本质上也是下载 release 资产装到同一位置） | `herdr update` |
| brew | 真实路径含 `/Cellar/herdr/` | `brew update && brew upgrade herdr` |
| mise | 真实路径含 `/mise/installs/herdr/` | `mise upgrade herdr` |
| nix | 真实路径含 `/nix/store/` | 按本机 nix 的方式更新（`nix profile upgrade` 等） |
| Windows 安装脚本 | `%LOCALAPPDATA%\Programs\Herdr\bin\herdr.exe` | 重新跑 `irm https://herdr.dev/install.ps1 | iex` |

安装命令：

```bash
# 官方安装脚本（Linux / macOS，默认装到 $HOME/.local/bin，可用 HERDR_INSTALL_DIR 改）
curl -fsSL https://herdr.dev/install.sh | sh

# Windows（PowerShell）
irm https://herdr.dev/install.ps1 | iex

# brew
brew install herdr

# mise
mise use -g herdr
```

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（`which` 的结果可能是软链，直接看会误判）：

```bash
readlink -f "$(command -v herdr)"
```

第二步，按**路径片段**对照上表：

| 真实路径片段 | 安装方式 |
| --- | --- |
| 含 `/Cellar/herdr/` | brew |
| 含 `/mise/installs/herdr/` | mise |
| 含 `/nix/store/` | nix |
| 是普通文件（软链已解析），且不在上面这些目录里 | 官方脚本 / 手动下载 |
| 解析后落在 `~/.local/bin/herdr` | 官方脚本（默认 `HERDR_INSTALL_DIR`） |

第三步，交叉验证一下包管理器（最可靠，路径判据对软链可能失效）：

```bash
brew list --versions herdr 2>/dev/null
mise ls herdr 2>/dev/null
```

两个都列出来说明机器上装过多份，用 `which -a herdr` 看 PATH 上到底生效的是哪个。

`herdr update` 自己也做同样的判断（它按当前二进制的真实路径识别 brew / mise / nix），
所以可以直接拿它的输出交叉验证。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

`herdr update` 会按检测结果分流：

- 直接安装（脚本 / 手动下载）：自己下载新二进制并原地替换当前这个文件。
- brew / mise / nix：**不自己动手**，只打印该用的命令，例如
  `brew update && brew upgrade herdr`、`mise upgrade herdr`。

所以更新时先跑一次 `herdr update` 看它给的是哪条命令，和上一步的检测结果核对；
不一致就按检测结果手动执行对应那一条。

```bash
herdr update                 # 直接安装：自行替换二进制
herdr update --handoff       # 尽量热交接，不停掉正在跑的终端
herdr channel set stable     # 切回正式通道
herdr channel set preview    # 预览通道（仅直接安装可用）
```

更新完确认版本：

```bash
herdr --version
which -a herdr
```

## agent 集成

`herdr` 自带 agent 集成（往各 agent 的目录里写状态回传 hook），和 CLI 更新是两回事：

```bash
herdr integration status                # 看各 agent 的集成状态
herdr integration install claude        # 装到支持的 agent 上
herdr integration uninstall claude
herdr integration install codex
```

支持的 target 以 `herdr integration install --help` 的候选列表为准
（claude、codex、opencode、pi、copilot、cursor、grok 等）。

agent skill 文件用 `herdr --skill` 打印（内容是给 agent 看的操作指南），
需要装成 skill 时把它写到对应 agent 的 skills 目录，别跟 `herdr integration install` 混为一谈。

## 注意

- **有会话在跑时 `herdr update` 会拒绝更新**，只打印
  `Herdr was not updated. Stop running Herdr sessions when ready, then run 'herdr update' again.`
  想不停掉终端就加 `--handoff`（live handoff，新 server 接管旧 pane，进程继续跑）：

  ```bash
  herdr update --handoff
  ```

  handoff 之后**原来挂在旧 session 上的客户端要重新连**：
  `herdr session attach <会话名>`。

- 更新完会提示 `installed herdr integrations need updating`，按提示重装对应集成：

  ```bash
  herdr integration install pi claude codex opencode
  ```

  集成版本落后时 agent 状态回传会不准。

- **客户端和 server 是两个进程。** `herdr` 只是个客户端，真正的 server 在后台常驻。
  更新完二进制，后台 server 还是旧版本；要让新版本生效需要重启 server
  （`herdr server stop` 后重开 `herdr`，或用 `herdr update --handoff` 试热交接）。
- agent skill 文件用 `herdr --skill` 打印，升级后内容会变；本地装成 skill 的要重新导出覆盖：
  `herdr --skill > ~/.claude/skills/herdr/SKILL.md`（路径按对应 agent 的 skills 目录调整）。
- 别直接跑裸 `herdr` 来「试探」有没有装——它是 TUI，会拉起界面挂住。用 `herdr --version`、
  `herdr --help`。
- 配置文件在 `~/.config/herdr/config.toml`；`herdr server reload-config` 让运行中的 server 重载，
  `herdr config reset-keys` 会先备份再恢复默认键位。
- `herdr server stop` 会停掉 server，正在跑的终端会被停掉——不是「关窗口」那种语义，别随手执行。
- 配置项 `HERDR_INSTALL_DIR` 决定脚本装到哪，`HERDR_ENV=1` 表示当前进程跑在 herdr 管理的 pane 里
  （agent 用它判断能不能调 `herdr` 控制会话）。

# Herdr App

- 命令：`herdr-app`
- 仓库地址：https://github.com/herdrdev/herdr-app
- 官网：https://herdr.dev
- 分发站：https://releases.herdr.dev/app/

Herdr 的**桌面客户端**（终端 + coding agent 的工作区窗口）。终端真正跑在后台的
herdr-core（`hc`）里，关掉窗口终端还活着。

CLI 部分（`herdr` 命令）是另一个东西，见 [Herdr CLI](herdr.md)。

## 分发方式（不走 GitHub Release）

这个应用**没有 GitHub Release**，二进制发布在自己的分发站上：

| 平台 | 资产 |
| --- | --- |
| Linux x86_64 | `herdr-app-linux-x86_64.tar.gz` |
| Linux aarch64 | `herdr-app-linux-aarch64.tar.gz` |
| macOS aarch64 | `herdr-app-macos-aarch64.tar.gz`（Intel Mac 不提供构建） |
| Windows x86_64 | `herdr-app-windows-x86_64.tar.gz` |

带校验和的清单在 https://releases.herdr.dev/app/latest.json ：

```json
{
  "version": "0.1.13",
  "core_min": "0.1.11",
  "targets": {
    "linux-x86_64": {
      "url": "https://releases.herdr.dev/app/0.1.13/herdr-app-linux-x86_64.tar.gz",
      "sha256": "77b0be72..."
    }
  }
}
```

`core_min` 是这个版本要求的核心（`hc`）最低版本。校验和就在这份清单里，没有单独的
`checksums.txt`。

## 安装方式清单

**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方安装脚本 | Linux 真实路径是 `~/.local/share/herdr-app/herdr-app`，`~/.local/bin/herdr-app` 是指向它的软链 | 重新跑 `curl -fsSL https://releases.herdr.dev/app/install.sh | sh` |
| macOS 手动拖拽 dmg | `~/Applications/Herdr.app` 存在，且 `~/.local/bin/herdr-app` 指向它 | 下载新 dmg 覆盖，或重跑安装脚本 |
| brew cask | `brew list --cask` 里有 `herdr-app` | `brew upgrade --cask herdr-app` |
| Windows 安装脚本 | `%LOCALAPPDATA%\Programs\herdr-app` | 重新跑 `irm https://releases.herdr.dev/app/install.ps1 | iex` |
| 源码构建 | 路径指向 git 仓库的构建产物 | 重新 `just build && just install` |

安装命令：

```bash
# Linux / macOS
curl -fsSL https://releases.herdr.dev/app/install.sh | sh

# Windows（PowerShell）
irm https://releases.herdr.dev/app/install.ps1 | iex

# macOS（Apple Silicon）
brew install --cask herdr-app
```

安装脚本做的事：

- **Linux**：应用放到 `~/.local/share/herdr-app/`，`~/.local/bin/herdr-app` 建软链，
  写 `~/.local/share/applications/herdr-app.desktop` 桌面项，
  图标放 `~/.local/share/icons/hicolor/scalable/apps/herdr-app.svg`，
  并跑 `update-desktop-database`。
- **macOS**：应用放到 `~/Applications/Herdr.app`（保留签名），`~/.local/bin/herdr-app`
  软链到 `Herdr.app/Contents/MacOS/Herdr`。
- 顺便装 **herdr-core**（`hc`）：只在 `~/.local/bin/herdr-core` 不存在时才装，
  已有的（自己的或包管理器装的）不动。

## 检测当前安装方式

```bash
readlink -f "$(command -v herdr-app)"
```

| 真实路径 / 特征 | 安装方式 |
| --- | --- |
| 软链指向 `~/.local/share/herdr-app/herdr-app` | 官方安装脚本（Linux） |
| 软链指向 `~/Applications/Herdr.app/Contents/MacOS/Herdr` | macOS（脚本或 dmg 拖拽，需再看下面一步） |
| 路径在 `/Applications/Herdr.app/...` | macOS，手动拖的 dmg |
| `brew list --cask` 里有 `herdr-app` | brew cask |

macOS 上脚本装和 dmg 拖拽装的**结果路径一样**，靠路径分不出来。脚本装会额外写桌面项
`~/.local/share/applications/herdr-app.desktop`，dmg 拖拽装的没有这个文件：

```bash
grep Exec ~/.local/share/applications/herdr-app.desktop 2>/dev/null
```

脚本写的 `Exec` 是 `~/.local/bin/herdr-app`（绝对路径），这个软链又指向 `Herdr.app`；
如果桌面项不存在、而 `~/Applications/Herdr.app` 在，那就是手动拖的 dmg。
实在分不出来就问用户。

## 更新

- **官方脚本装的**：重跑一次安装脚本即可，脚本幂等，会把应用换成新版本：

  ```bash
  curl -fsSL https://releases.herdr.dev/app/install.sh | sh
  ```

- **brew cask 装的**：

  ```bash
  brew upgrade --cask herdr-app
  ```

- **dmg 拖拽装的**：下载 https://releases.herdr.dev/app/Herdr.dmg ，把新的
  `Herdr.app` 拖到 `~/Applications`（或 `/Applications`）覆盖旧的。

- **应用自己也管更新**：正式版会在后台检查并下载新版本（Settings → Updates 里能看到），
  下载的新应用放在正在运行的这个旁边，**退出并重开应用后才切换过去**。
  所以「更新了但版本没变」通常是还没退出重开。
  源码构建的版本不自动更新。

更新完确认版本：

```bash
herdr-app --version
```

## herdr-core（hc）

终端真正跑在 core 里，core 是一个独立二进制：

- 装在 `~/.local/bin/herdr-core`，同目录有个 `hc` 软链（有些窗口管理器自带同名 `hc`，
  那种情况下不建软链）。
- 应用安装脚本只在 core 缺失时装它；装好后由应用负责保持 core 是新的。
- 手动装 / 更新 core：

  ```bash
  curl -fsSL https://releases.herdr.dev/core/install.sh | sh
  hc update            # 只更新 core
  ```

- 清单在 https://releases.herdr.dev/core/latest.json ，资产是
  `herdr-core-<平台>-<架构>.tar.gz`。

## 命令行用法

应用开着的时候，`herdr-app` 也能从终端指挥它：

```bash
herdr-app ls         # 窗口里的 spaces、tabs、terminals
herdr-app status     # 窗口现在显示什么，有没有什么错
herdr-app commands   # 所有命令及其参数
herdr-app shot       # 存一张窗口截图
herdr-app --skill    # 打印给 agent 用的操作指南
```

## 注意

- Linux 需要有桌面会话和可用的 Vulkan 驱动；没有图形环境就装不了。
- Intel Mac 不提供构建，只能自己从源码编译。
- **关掉窗口不会停终端**：终端在 core 里继续跑。重开窗口会恢复布局，但原来的进程
  不会复活（重启机器或 core 崩溃后，保留的 pane 拿到的是新 shell，不是老进程）。
- macOS 上 dmg 拖拽装和脚本装都会往 `~/Applications` 放，注意别装两份。
- 应用会做后台更新检查并**收集使用遥测**（正式版默认开启），在
  Settings → Privacy 里关掉。
- Mods 是受信任的代码，不是沙箱扩展，装别人的 mod 前先看仓库里的 security 文档。

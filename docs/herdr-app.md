# Herdr App（桌面客户端）

- 命令：`herdr-app`
- 仓库地址：https://github.com/herdrdev/herdr-app
- 官网：https://herdr.dev
- 下载页：https://herdr.dev/download/

Herdr 的**桌面客户端**（GUI）。注意它和 `herdr` TUI 是两个东西，见 `docs/herdr.md`。

组成：

| 组件 | 仓库 | 命令 | 说明 |
| --- | --- | --- | --- |
| Herdr App | https://github.com/herdrdev/herdr-app | `herdr-app` | 桌面 GUI 客户端 |
| Herdr Core | https://github.com/herdrdev/herdr-core | `herdr-core` / `hc` | 真正跑终端和 agent 的后台核心 |

App 会管理 Core 的版本：Core 没装时 App 会提示装，装好之后由 App 负责更新。

## 资产与安装

**GitHub Release 上没有桌面版资产**，全部走自建发布站 `https://releases.herdr.dev`。

清单（带 sha256）：

| 组件 | 清单 |
| --- | --- |
| App | https://releases.herdr.dev/app/latest.json |
| Core | https://releases.herdr.dev/core/latest.json |

App 清单长这样（`targets` 里按平台给 `url` + `sha256`）：

```json
{
  "version": "0.1.16",
  "core_min": "0.1.11",
  "targets": {
    "linux-x86_64": {
      "url": "https://releases.herdr.dev/app/0.1.16/herdr-app-linux-x86_64.tar.gz",
      "sha256": "282d2632..."
    }
  }
}
```

### Linux

官方安装脚本（装到用户目录，**不需要 sudo**）：

```bash
curl -fsSL https://releases.herdr.dev/app/install.sh | sh
```

装完：

| 路径 | 内容 |
| --- | --- |
| `~/.local/share/herdr-app/` | App 本体（`herdr-app` 可执行文件 + `herdr-app.svg` 图标） |
| `~/.local/bin/herdr-app` | 指向 App 的软链 |
| `~/.local/share/applications/herdr-app.desktop` | 应用启动器（GNOME 里叫 “Herdr”） |
| `~/.local/share/icons/hicolor/scalable/apps/herdr-app.svg` | 图标 |

脚本还会顺手装 Core（**只在 `~/.local/bin/herdr-core` 不存在时**），
Core 有自己的安装脚本 `https://releases.herdr.dev/core/install.sh`，
装到 `~/.local/bin/herdr-core`，并额外建一个 `~/.local/bin/hc` 软链
（系统里已经有 `hc` 就不建——有些窗口管理器自带这个命令）。

### macOS

- Apple silicon：`https://releases.herdr.dev/app/Herdr.dmg`，拖进 `~/Applications`（或 `/Applications`）；
  或者跑同一个 `install.sh`，装到 `~/Applications/Herdr.app`。
- **Intel Mac 不支持**，只能自己从源码编译。

### Windows

```powershell
powershell -c "irm https://releases.herdr.dev/app/install.ps1 | iex"
```

## 检测当前安装方式

```bash
readlink -f "$(command -v herdr-app)"
```

| 真实路径片段 | 安装方式 |
| --- | --- |
| `/.local/share/herdr-app/` | 官方安装脚本（Linux） |
| `/Applications/Herdr.app/` 或 `~/Applications/Herdr.app/` | 官方安装脚本或 dmg（macOS） |
| 其它 | 手动下载 tarball / 包管理器 |

## 更新

**App 更新自己**（打开时检查，有新版就提示）；也可以重跑安装脚本：

```bash
curl -fsSL https://releases.herdr.dev/app/install.sh | sh
```

重跑脚本会替换 `~/.local/share/herdr-app/`，桌面项和图标也会一起刷新。

Core 用自己的更新命令：

```bash
hc update                    # 下载最新版本换掉当前的
hc update --version 0.1.21   # 至少要这个版本
hc update --from <二进制>    # 换成指定的构建
```

`hc update` 换的是**正在跑的** Core，官方说法是「terminals keep running」，
不用像 `herdr update`（TUI）那样先退出所有会话。

## 配置

| 位置 | 内容 |
| --- | --- |
| `~/.config/herdr-app/` | App 配置，另有 `AGENTS.md`（告诉 agent 它能改什么） |
| `~/.config/herdr-core/config.toml` | Core 配置（`notes = false` 可以关掉自动写笔记） |

### 终端字体

`~/.config/herdr-app/settings.json` 里的 `font` / `font_size` / `font_weight` / `ui_font`
控制终端字体，改完**即时生效，不用重启**：

```json
{
  "font": "Maple Mono CN",
  "font_size": 14,
  "font_weight": 400,
  "ui_font": "Maple Mono CN"
}
```

- `font` / `font_size` / `font_weight`：终端里的文字；
- `ui_font`：面板（mods）的文字，本机也用 Maple Mono CN，界面和终端同一个字体。

本机用 Maple Mono CN 的原因和系统级 fontconfig 配置见 [docs/fonts.md](fonts.md)。

几个坑：

- **只有布局没自己指定字体时，`settings.json` 才生效**。`main` 布局不指定，用 `settings.json` 的；
  `terminals` 布局只定 `font_size: 15`（字体仍继承 `settings.json`）；
  `win98` 布局自带像素字体（`PxPlus IBM VGA 8x16`），不跟随。
- **新装的字体要重启 App 才认**。App 只在启动时扫描字体目录，
  装了新字体只改 `settings.json` 不重启的话不会生效。
- **Herdr 没有单独的中文字体设置项**，终端字体只能整体指定，中文靠字体自带字形
  （所以要选自带 CJK 字形的字体，见 [docs/fonts.md](fonts.md)）。
- `font_weight` 默认 500（Medium），400 是 Regular，700 是 Bold。
- `font_ligatures` 默认关闭（编程连字），要开设 `true`。Maple Mono 带连字那套
  （`MapleMono-CN-*`，不带连字的是 `MapleMonoNL-CN-*`）可以显示 `=>`、`!=` 之类连字。
- **字体家族名 v7.9 起改过名**：老版本是 `Maple Mono Normal CN`，现在是 `Maple Mono CN`，
  写错名字会静默回退到 JetBrains Mono，看起来像「设置没生效」。
  用 `fc-match "Maple Mono CN"` 核对。

## 注意

- **App、TUI（`herdr`）、Core（`hc`）是三个独立的安装物**，装在哪、怎么更新各不相同，别混：
  - App：装到 `~/.local/share/herdr-app`，更新靠 App 自己或重跑 `install.sh`；
  - TUI：见 `docs/herdr.md`，更新用 `herdr update`；
  - Core：装到 `~/.local/bin/herdr-core`，更新用 `hc update`。
- App 的桌面项 `Exec=` 写的是**绝对路径** `~/.local/bin/herdr-app`，改成相对路径在图形会话里会打不开。
- 安装脚本自己会校验 sha256（用的是上面那份 `latest.json`），
  下坏了会报 `download is damaged (checksum mismatch)`。
- 手动核对也可以：

  ```bash
  curl -s https://releases.herdr.dev/app/latest.json | python3 -c \
    "import sys,json;d=json.load(sys.stdin);print(d['version'],d['targets']['linux-x86_64']['sha256'])"
  sha256sum ~/.local/share/herdr-app/herdr-app.tar.gz   # 手动下的包才需要
  ```

- 桌面版**没有 Flatpak / rpm / deb 包**（至少 2026-11-18 没有），只有 tarball 和官方脚本；
  Flathub 上也搜不到。
- App 第一次启动可能要求 Core 版本不低于 `core_min`（0.1.16 要求 ≥ 0.1.11），
  Core 太旧会提示升级。

# Ghostty

- 仓库地址：https://github.com/ghostty-org/ghostty
- 官网：https://ghostty.org
- 社区打包仓库（Ubuntu/Debian）：https://github.com/mkasberg/ghostty-ubuntu

Zig 编写、GPU 加速的终端模拟器，界面用 GTK4 + libadwaita 原生组件，支持 Linux、macOS，
Windows 还在开发中。

**关键前提：上游官方只发布 macOS 的预编译二进制。** Linux 拿不到官方二进制，必须走
发行版官方包、社区打包（`.deb` / PPA / COPR / Nix / snap）或自己编译。

## Release 资产

上游 GitHub Releases 里**只有一个 `tip` 预发布**（assets 14 个：`Ghostty.dmg`、
`ghostty-macos-universal*.zip`、`ghostty-source.tar.gz`、`.wasm`、`.xcframework` 等），
**没有稳定版 Release 对象**——稳定版只以 git tag 存在（`v1.4.0`、`v1.3.1`、`v1.2.x`……）。
所以：

- 查最新稳定版版本号用 tag：`gh api repos/ghostty-org/ghostty/tags --jq '.[].name'`
- 别用 `releases/latest` 找 Linux 安装包，那个接口 404；
- 别下载 `tip` 的 `ghostty-source.tar.gz` 当稳定版源码，它是滚动开发快照。

Linux 的 `.deb` 不在上游，而在社区仓库 `mkasberg/ghostty-ubuntu` 的 Releases 里。

## 安装（按平台）

### Linux

#### 社区 deb（Debian / Ubuntu）

社区仓库 <https://github.com/mkasberg/ghostty-ubuntu> 提供打包好的 `.deb`，资产命名：

```
ghostty_<版本>-0~ppaN_<架构>_<发行版代号>.deb
```

发行版代号取 Ubuntu 的 `26.04` / `26.10` / `24.04` 或 Debian 的 `trixie` / `forky`，
架构为 `amd64` / `arm64`。例：

```bash
curl -LO https://github.com/mkasberg/ghostty-ubuntu/releases/download/1.4.0-0-ppa2/ghostty_1.4.0-0.ppa2_amd64_trixie.deb
sudo apt-get install -y ./ghostty_1.4.0-0.ppa2_amd64_trixie.deb
```

该仓库还提供一键脚本（内部逻辑同上：探测发行版 → 找最新 release 的 `.deb` → `apt-get install`）：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/mkasberg/ghostty-ubuntu/HEAD/install.sh)"
```

也可以走 Launchpad PPA，之后就能被 `apt upgrade` 带上：

```bash
sudo add-apt-repository ppa:mkasberg/ghostty-ubuntu
sudo apt update
sudo apt install ghostty
```

deb 里装的东西：`/usr/bin/ghostty`、桌面入口 `/usr/share/applications/com.mitchellh.ghostty.desktop`
（`Exec=ghostty --gtk-single-instance=true`，图标 `com.mitchellh.ghostty`；1.3.1 的 `Exec`
是绝对路径 `/usr/bin/ghostty`，1.4.0 改成相对命令）、
hicolor 图标、`/usr/share/terminfo/x/xterm-ghostty`（`TERM=xterm-ghostty`）、
zsh 补全 `_ghostty`、systemd 用户单元 `app-com.mitchellh.ghostty.service`。

运行依赖：GTK4（`libgtk-4-1`）、libadwaita（`libadwaita-1-0`）、fontconfig、freetype、
harfbuzz、EGL、Wayland client、X11。

#### 发行版官方包

| 发行版 | 安装命令 |
| --- | --- |
| Ubuntu 26.04+ / Debian | `sudo apt install ghostty`（官方源版本偏旧，Ubuntu 26.04 首发是 1.3.0） |
| Arch | `sudo pacman -S ghostty` |
| Fedora | `sudo dnf copr enable scottames/ghostty && sudo dnf install ghostty` |
| openSUSE | `sudo zypper in ghostty` |
| Alpine | `apk add ghostty` |
| Gentoo | `emerge -av ghostty` |
| Nix / NixOS | nixpkgs 里包名 `ghostty`（Linux 版）；macOS 用 `ghostty-bin` |
| snap | `snap install ghostty --classic` |

NixOS 配置：

```nix
environment.systemPackages = [ pkgs.ghostty ];
users.users.somebody.packages = [ pkgs.ghostty ];
```

非 NixOS 系统用 nixpkgs 的 `ghostty` **会启动失败**（找不到 OpenGL 驱动库路径），
需要 nixGL / nix-gl-host 之类的包装。

#### Flatpak

Flathub 上**没有**官方 `com.mitchellh.ghostty`（截至 2026-11 未收录），
`flatpak remote-ls flathub | grep -i ghostty` 为空。本项目约定优先 flatpak，
但 Ghostty 只能退回到上面的 deb / 发行版包。

#### 源码编译

```bash
git clone https://github.com/ghostty-org/ghostty
cd ghostty
zig build -Doptimize=ReleaseFast
```

打包自己的平台看仓库里的 `PACKAGING.md`。

### macOS

- 官网下载 `.dmg`（`https://ghostty.org/download`），官方签名并公证，拖进 `/Applications`。
- Homebrew：`brew install --cask ghostty`（装的就是官方 dmg 的再打包）。
- Nix：`pkgs.ghostty-bin`。

### Windows

官方尚未提供 Windows 构建，处于开发中。

## 检测当前安装方式

```bash
command -v ghostty            # 路径
dpkg -l ghostty               # 系统包版本
apt-cache policy ghostty       # 候选版本来自哪个源
flatpak list | grep -i ghostty
snap list | grep -i ghostty
```

判据：

- `/usr/bin/ghostty` + `dpkg -l ghostty` 有记录 → 系统包（deb / PPA 装的）。
  再看 `apt-cache policy ghostty` 的候选版本来源：`ppa.launchpadcontent.net/mkasberg/...`
  是 PPA 装的，其余（`debian/...`）是发行版官方源装的。
- `~/.local/bin/ghostty`（软链或自编译）→ 手动装 / 自编译。
- `flatpak list` / `snap list` 有记录 → 对应沙箱包。
- nix profile 目录里出现 → nix 装的。

## 更新

**沿用当初的安装方式**，不要换：

- deb 直接装的（下载 `.deb` 装的）：从社区仓库 Releases 下载对应发行版代号的新 `.deb`，然后

  ```bash
  sudo apt-get install -y ./ghostty_<新版本>-0~ppaN_<架构>_<代号>.deb
  ```

  这是升级安装，会替换旧版并保留配置。
- PPA 装的：`sudo apt update && sudo apt install ghostty`（或 `sudo apt upgrade ghostty`）。
- 发行版官方源装的：用该发行版的原生命令（`pacman -Syu ghostty`、`dnf install ghostty`……）。
- nix 装的：`sudo nix profile upgrade`。
- snap 装的：`sudo snap refresh ghostty`。
- macOS dmg / brew cask：重新下载拖进 `/Applications`，或 `brew upgrade --cask ghostty`。

查社区仓库最新版本：

```bash
gh api repos/mkasberg/ghostty-ubuntu/releases/latest --jq '.tag_name, [.assets[].name]'
```

## 配置

配置文件在 `~/.config/ghostty/config`（一行一条 `key = value`）。
辅助命令：

```bash
ghostty +show-config        # 打印生效配置
ghostty +validate-config    # 校验配置语法
ghostty +list-keybinds      # 列出快捷键
```

配置项与默认值见 <https://ghostty.org/docs/config/reference>。

## 注意

- **本机状态（2026-11-07 核实）**：Debian trixie / amd64，`/usr/bin/ghostty`。此前装的是
  `1.3.1-6~trixie`（Debian 官方源打包格式，版本号带 `-6~trixie`，不是社区 deb 的 `-0~ppa2`），
  2026-11-07 直接下载社区仓库 `mkasberg/ghostty-ubuntu` 的
  `ghostty_1.4.0-0.ppa2_amd64_trixie.deb` 升级到 `1.4.0-0~ppa2`，`ghostty --version` 报 `Ghostty 1.4.0`。
- **从官方源包升到社区 deb 可以直接覆盖**：`sudo apt install ./ghostty_1.4.0-0.ppa2_amd64_trixie.deb`
  会把 `1.3.1-6~trixie` 替换掉，postinst 顺带重建 systemd 用户单元软链
  `/etc/systemd/user/graphical-session.target.wants/app-com.mitchellh.ghostty.service`。
  升级后菜单里打不开就退出再登录一次，并刷新图标缓存：
  `sudo gtk-update-icon-cache -f -t /usr/share/icons/hicolor`。
- **本机的 apt 源里已经没有 ghostty 条目**（`/etc/apt/sources.list.d/` 下没有），
  所以 `apt upgrade` 不会更新 ghostty，`apt-cache policy ghostty` 只能看到已安装版本。
  要用 apt 路线更新，先 `sudo add-apt-repository ppa:mkasberg/ghostty-ubuntu`；
  要么就直接装新 `.deb`。
- 版本号里的 `-0~ppaN` 是社区重新打包的次数，不是不同分支；看到 `1.4.0-0.ppa2` 和
  `1.3.1-0~ppa2` 属于同一条发布线。
- **不要混装官方源和社区 PPA**：官方源版本更旧，`apt` 升级时可能把版本降回去
  （需要 `apt-mark hold ghostty` 或直接用 `.deb` 升级路线）。
- 上游仓库名是 `ghostty-org/ghostty`（不是 `mitchellh/ghostty`）；社区打包仓库是
  `mkasberg/ghostty-ubuntu`，两者不要混淆。
- Linux 下 Ghostty 依赖 GTK4 与 libadwaita，老发行版（如 Debian 12）可能缺
  `libadwaita-1-0` 装不上，这时换发行版官方源或源码编译。
- 用 Nix 装在非 NixOS 上起不来是已知问题（OpenGL 库路径），不是配置错。
- 桌面入口是 `com.mitchellh.ghostty.desktop`。**1.3.1 里写的是绝对路径**
  （`TryExec=/usr/bin/ghostty`、`Exec=/usr/bin/ghostty --gtk-single-instance=true`），
  **1.4.0 改成了相对命令**（`TryExec=ghostty`、`Exec=ghostty --gtk-single-instance=true`），
  依赖 PATH 里有 `/usr/bin`（一般都有）。`Icon=com.mitchellh.ghostty` 走 hicolor 主题，
  图标由 deb 装进 `/usr/share/icons/hicolor/*/apps/`；菜单图标空白时跑
  `sudo gtk-update-icon-cache -f -t /usr/share/icons/hicolor`。
- 用 `apt-get install ./xxx.deb` 升级时，debconf 会报「对话框界面将不能在哑终端上运行」
  一类的 front-end 警告，**这是正常的**，包本身装成功，不影响结果。
- 新版本会重新生成 systemd 用户单元软链 `/etc/systemd/user/graphical-session.target.wants/
  app-com.mitchellh.ghostty.service`，升级后如果菜单里打不开，退出再登录一次即可。

- **本机状态（2026-11-12 核实）**：Debian trixie / amd64，已装 `1.4.0-0~ppa2`（社区 PPA 打包）。
  上游最新稳定 tag 仍是 `v1.4.0`，社区仓库最新 release 也还是 `1.4.0-0-ppa2`，**已是最新，无需更新**。
  Debian 官方源那个 `1.4.0-2~trixie` 是官方打包的同版本，不要混装（见上面「不要混装官方源和社区 PPA」）。

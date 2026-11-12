# VS Code

- 仓库地址：https://github.com/microsoft/vscode
- 官网：https://code.visualstudio.com

微软的跨平台代码编辑器（Electron/Chromium + Code-OSS）。Linux、macOS、Windows 都有。

## 不走 GitHub Release

`microsoft/vscode` 的 release 里**只有源码 tag，没有任何二进制资产**（`gh api repos/microsoft/vscode/releases/latest --jq '.assets'` 返回空数组）。
二进制由微软自己的更新服务器 `update.code.visualstudio.com` 和 apt / rpm 仓库分发，别用 `gh release download` 找安装包。

版本对应关系：

| 渠道 | 版本写法 | 例 |
| --- | --- | --- |
| GitHub tag / 官网版本 | 上游版本号 | `1.145.2` |
| apt / rpm 包版本 | 上游版本号 + 构建号 | `1.145.2-1793945276` |
| insiders 更新接口返回的 `name` | 上游版本号 + `-insider` | `1.146.0-insider` |
| insiders 包文件名 / apt 版本 | `code-insiders_<上游版本号>_<构建号>_amd64.deb` | `code-insiders_1.146.0-1793993931_amd64.deb` |

查最新版本用官网的更新接口（返回 JSON，含下载地址和 sha256）：

```bash
curl -s "https://update.code.visualstudio.com/api/update/linux-deb-x64/stable/latest"
```

## 更新服务器 URL 规则

下载地址的格式是 `https://update.code.visualstudio.com/{版本}/{平台标识}/{渠道}`，
版本可省（用 `latest`），渠道是 `stable` 或 `insider`（**单数**，写 `insiders` 返回 404）。

```bash
# 最新稳定版 deb（直接返回 302 到真实文件）
curl -IL -o /dev/null -w '%{url_effective}\n' \
  "https://update.code.visualstudio.com/latest/linux-deb-x64/stable"
```

| 平台标识 | 产物 | 适用架构 |
| --- | --- | --- |
| `linux-deb-x64` / `linux-deb-arm64` / `linux-deb-armhf` | `.deb` | x86_64 / arm64 / armhf |
| `linux-rpm-x64` / `linux-rpm-arm64` / `linux-rpm-armhf` | `.rpm` | 同上 |
| `linux-x64` / `linux-arm64` / `linux-armhf` | `.tar.gz`（免安装） | 同上 |
| `darwin-universal` / `darwin` / `darwin-arm64` | `.zip` | macOS |
| `win32-x64-user` / `win32-arm64-user` | 用户级安装器 `.exe` | Windows |
| `win32-x64` / `win32-arm64` | 系统级安装器 `.exe` | Windows |
| `win32-x64-archive` / `win32-arm64-archive` | 免安装 `.zip` | Windows |

Windows 的 `.exe` 是安装器不是压缩包，用 `unzip` 解不开。

## 安装方式清单

装的时候选了哪种，以后更新就用哪种。

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| Microsoft apt 仓库（Debian/Ubuntu） | `dpkg -L code` 里是 `/usr/share/code/`，apt 源里有 `packages.microsoft.com/repos/code` | `sudo apt install --only-upgrade code` |
| Microsoft rpm 仓库（Fedora/RHEL/openSUSE） | `rpm -ql code` 里是 `/usr/share/code/`，yum 源里有 `packages.microsoft.com/yumrepos/code` | `sudo dnf upgrade code` |
| 手动装 `.deb` / `.rpm` | 同上，但没有配 apt/rpm 源 | 重新下载新包覆盖安装 |
| 免安装 tar.gz | 真实路径含 `VSCode-linux-` 这样的解压目录 | 下载新 tar.gz 整目录替换 |
| Flatpak | `flatpak list` 里有 `com.visualstudio.code` | `flatpak update com.visualstudio.code` |
| Snap | `snap list` 里有 `code` | `sudo snap install code --classic`（或 `snap refresh code`） |
| Arch AUR | `pacman -Qo "$(command -v code)"` 指向 `.local/share/aur` | `paru -Syu code` / `yay -Syu code` |
| macOS dmg / zip | `/Applications/Visual Studio Code.app` | 下载新 dmg 覆盖 `/Applications` 里的 `.app` |
| macOS Homebrew cask | `brew list --cask` 里有 `visual-studio-code` | `brew upgrade --cask visual-studio-code` |
| Windows 安装器 | `%LOCALAPPDATA%\Programs\Microsoft VS Code\` | 运行新安装器覆盖安装 |
| Windows 免安装 zip | `code.exe` 同级有 `data` 文件夹 | 下载新 zip 解压替换 |

### Microsoft apt 仓库（Debian/Ubuntu）

先加源（需要 sudo）：

```bash
sudo install -D -m 0755 /dev/stdin /etc/apt/keyrings/packages.microsoft.gpg \
  < <(curl -fsSL https://packages.microsoft.com/keys/microsoft.asc)
echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" \
  | sudo tee /etc/apt/sources.list.d/vscode.list
sudo apt update
sudo apt install code
```

同一个仓库里也带 insiders 版，包名是 `code-insiders`：

```bash
sudo apt install code-insiders
```

`code` 和 `code-insiders` 可以共存，命令名分别是 `code` 和 `code-insiders`，
配置和扩展目录也是分开的（见下面「数据目录」）。

### 手动装 .deb

没有配 apt 源也能装，但依赖要先补齐：

```bash
sudo apt install ./code_1.145.2-1793945276_amd64.deb
```

装完 dpkg 里会记着版本，但**不在任何 apt 源里**，以后 `apt upgrade` 不会管它，
要更新得自己重新下包覆盖。

### 免安装 tar.gz

不用 sudo，解压到自己的目录。注意**文件名里没有版本号，只有构建号**
（`code-stable-x64-1793945155.tar.gz`），版本号从 `Content-Disposition` 头里看；
解压出的顶层目录叫 `VSCode-linux-x64`（arm64 是 `VSCode-linux-arm64`）：

```bash
mkdir -p ~/.local/opt
tar -xzf code-stable-x64-1793945155.tar.gz -C ~/.local/opt
ln -sf ~/.local/opt/VSCode-linux-x64/bin/code ~/.local/bin/code
```

图标在解压目录的 `resources/app/resources/linux/code.png`；
**tar.gz 里不含 `.desktop` 文件**（实测整个包 0 个 `.desktop`），桌面入口要自己写，
放到 `~/.local/share/applications/`：

```ini
[Desktop Entry]
Name=Visual Studio Code
Comment=Code Editing. Redefined
Exec=/home/<用户>/.local/opt/VSCode-linux-x64/code %F
Icon=vscode
Type=Application
Terminal=false
StartupNotify=true
StartupWMClass=com.microsoft.VSCode
Categories=TextEditor;Development;IDE;
MimeType=text/plain;inode/directory;
```

`Exec` 写绝对路径（桌面环境的 PATH 里通常没有 `~/.local/bin`），`Icon=vscode` 依赖图标主题缓存，
把 `code.png` 拷到 `~/.local/share/icons/hicolor/512x512/apps/vscode.png` 后跑：

```bash
update-desktop-database ~/.local/share/applications/
gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor
```

`StartupWMClass=com.microsoft.VSCode` 来自 `product.json` 的 `linuxDesktopName`
（Electron 的 `app.setDesktopName()` 决定 WM_CLASS，写错了任务栏图标会跟窗口分家）；
图标名 `vscode` 来自同一处的 `linuxIconName`。

版本也可以从解压后的 `resources/app/package.json` 里读（`"version": "1.145.2"`）。

### Flatpak

Flathub 上有官方认可的包 `com.visualstudio.code`：

```bash
flatpak install flathub com.visualstudio.code
flatpak run com.visualstudio.code
```

想在终端直接用 `code`，建个 wrapper（wrapper 放在别处，不要放进解包目录里）：

```bash
mkdir -p ~/.local/bin
cat > ~/.local/bin/code <<'EOF'
#!/bin/sh
exec flatpak run com.visualstudio.code "$@"
EOF
chmod +x ~/.local/bin/code
```

Flatpak 版拿不到宿主机的 `~/.vscode/extensions`，扩展装在 flatpak 自己的沙箱里，
和 deb 版 / tarball 版的扩展不通用；版本也通常落后几个版本。

### macOS

官网下载 dmg（universal）拖进 `/Applications`；命令行入口是
`/Applications/Visual Studio Code.app/Contents/Resources/app/bin/code`，软链到 `~/.local/bin/code` 才能在终端用。

Homebrew：`brew install --cask visual-studio-code`，更新 `brew upgrade --cask visual-studio-code`。

### Windows

- 用户安装器 `win32-x64-user`：装到 `%LOCALAPPDATA%`，不需要管理员权限，会自动更新。
- 系统安装器 `win32-x64`：装到 `Program Files`，需要管理员权限。
- 免安装 zip `win32-x64-archive`：解压后**新建一个空的 `data` 文件夹**，VS Code 会进入 Portable 模式，
  用户数据、扩展都放在这个 `data` 里，不会污染系统。

## 检测当前安装方式

```bash
readlink -f "$(command -v code)"
dpkg -S "$(readlink -f "$(command -v code)")" 2>/dev/null
```

| 现象 | 安装方式 |
| --- | --- |
| 真实路径是 `/usr/share/code/bin/code`，且 `dpkg -S` 能查到 `code` 包 | Microsoft apt 仓库 |
| `rpm -qf` 能查到 `code` 包 | Microsoft rpm 仓库 |
| 真实路径含 `VSCode-linux-` 这样的解压目录 | 免安装 tar.gz |
| `flatpak list --app \| grep com.visualstudio.code` 有输出 | Flatpak |
| `snap list \| grep code` 有输出 | Snap |
| 真实路径在 `~/.local/share/aur/` 或 AUR 的 build 目录 | AUR |
| macOS 上 `/Applications/Visual Studio Code.app` 存在，且 `brew list --cask` 里有它 | Homebrew cask |
| Windows 上 `%LOCALAPPDATA%\Programs\Microsoft VS Code\` 存在 | 安装器（用户级或系统级） |
| Windows 上 `code.exe` 同级有 `data` 文件夹 | 免安装 zip |

判断不出来就问用户，不要猜、不要擅自换方式重装。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的后果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

apt 仓库装的（本项目常见）：

```bash
sudo apt update
sudo apt install --only-upgrade code
```

`apt update` 后候选版本通常就是最新稳定版；仓库比官网晚几小时到几天也正常，
急着要最新版就下 `https://update.code.visualstudio.com/latest/linux-deb-x64/stable` 的 deb 手动装
（但装完 dpkg 版本会高于仓库里的，下次 `apt upgrade` 可能被提示降级）。

其余方式：

```bash
# rpm 仓库
sudo dnf upgrade code

# Flatpak
flatpak update com.visualstudio.code

# Snap
sudo snap install code --classic     # 首次；已装则是 snap refresh code

# tar.gz：下载新包，解压替换整个目录，软链路径不变就不用重连

# macOS dmg / brew
brew upgrade --cask visual-studio-code
```

验证版本：

```bash
code --version
```

输出三行：版本号、commit hash、架构（`x64` / `arm64`）。

## 数据目录

| 目录 | 内容 | insiders 对应目录 |
| --- | --- | --- |
| `~/.config/Code/` | 用户设置、状态 | `~/.config/Code - Insiders/` |
| `~/.vscode/extensions/` | 扩展 | `~/.vscode-insiders/extensions/` |
| `~/.config/Code/User/` | `settings.json`、`keybindings.json` | `~/.config/Code - Insiders/User/` |

升级只换程序本体，这几个目录不动，扩展和设置都保留。

## 注意

- **本机当前状态**：Debian trixie / amd64，Microsoft apt 仓库装的 `code` 包
  （`/usr/share/code/`，apt 源 `/etc/apt/sources.list.d/vscode.list` 指向
  `packages.microsoft.com/repos/code`）。2026-11-08 从 `1.143.1-1792691907` 升到 `1.145.2-1793945276`，
  用的是 `sudo apt install --only-upgrade code`。
- **别从 GitHub Release 下**：`microsoft/vscode` 的 release 里没有二进制资产，`gh release download` 会失败。
- **版本号写法**：官网 / GitHub tag 是 `1.145.2`，apt 包版本是 `1.145.2-1793945276`，
  后半段是微软自己的构建号。两者对不上时以 dash 前面那段为准。
- **渠道名是 `insider` 不是 `insiders`**：`latest/linux-deb-x64/insiders` 返回 404，
  `latest/linux-deb-x64/insider` 才有效；`api/update/...` 接口同样用 `insider`。
- **Microsoft 源连接不稳**：`packages.microsoft.com` 偶尔 TLS 握手失败（curl 报 exit 35），
  重试即可；`apt update` 报 `Hash Sum mismatch` 通常也是这个原因，多跑一次。
- **apt 安装时的 debconf 警告是正常的**：`debconf: 无法初始化前端界面` / `Readline` /
  `Teletype` 这类提示只是说没法弹交互界面，包本身装成功，不影响结果。
- **别用 root 跑 GUI**：以 root 启动会弹 "You are trying to start Code as a super user which is not recommended"，
  确实要跑就加 `--no-sandbox --user-data-dir=<目录>`。
- **降级**：`apt install code=1.143.1-1792691907` 可以退回旧版本，但 apt 源里旧包会被清掉，
  降级前先确认 `apt-cache madison code` 里还有那一版。
- **snap 版沙箱**：扩展目录在 `~/snap/code/`，访问宿主 `~/.config` 受限，
  装了 snap 版就别再装 deb 版，两个 `code` 会抢 PATH。
- **Flatpak 版扩展不通用**：扩展装在沙箱内，和 deb / tar.gz 版互不可见。
- **macOS 首次打开被拦**：dmg 下载后需要右键 → 打开，或
  `xattr -dr com.apple.quarantine "/Applications/Visual Studio Code.app"`。
- **Windows 免安装版**：解压后必须新建空的 `data` 文件夹才会进入 Portable 模式。
- **命令行用法**：`code <目录>` 打开目录、`code --install-extension <id>` 装扩展、
  `code --list-extensions` 列扩展、`code --tunnel` 起远程隧道（另有独立的 `code-tunnel` 可执行文件）。

- **本机状态（2026-11-12 核实）**：Debian 13 (trixie) / amd64，**手动装 deb** 的
  （`apt-cache policy code` 候选版本与已装版本相同、优先级 100，说明没配 Microsoft apt 源）。
  本次从 `https://update.code.visualstudio.com/latest/linux-deb-x64/stable` 下载
  `code_1.146.0-1794340865_amd64.deb`（291 MB），sha256 `f83565f7f11a60b256807fe2216da8405f9728f193754959695242eecd148992`，
  用 `sudo apt install ./...deb` 从 1.145.2 升到 1.146.0。
  官网更新接口 `https://update.code.visualstudio.com/api/update/linux-deb-x64/stable/latest`
  返回的 JSON 里带下载地址和 sha256，下载后用它校验。

# 微信 Linux 版（WeChat for Linux）

- 官网：<https://linux.weixin.qq.com>
- 官方 Linux 版从 2024 年开始提供，当前版本 4.1.13（deb 内部版本号 4.1.13.26）
- Flatpak 应用 ID：`com.tencent.WeChat`

**先说结论：现在不需要第三方客户端。** 腾讯自己出了 Linux 版，功能和 Windows/Mac 版基本对齐。
下面列的「第三方方案」只剩存档价值，新装请直接用官方版。

## 官方资产

官网只提供直链下载，**没有官方 apt / dnf 源**。下载链接里不含版本号，是覆盖式更新：

| 架构 | deb | rpm | AppImage |
| --- | --- | --- | --- |
| x86_64 | `https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.deb` | 同目录 `.rpm` | `WeChatLinux_x86_64.AppImage` |
| arm64 | `WeChatLinux_arm64.deb` | `WeChatLinux_arm64.rpm` | `WeChatLinux_arm64.AppImage` |
| loongarch64 | `WeChatLinux_LoongArch.deb` | 无 | 无 |

deb 内部信息（实测）：

- 包名 `wechat`（**不是** `weixin`），`Provides: wechat`
- 主体装在 `/opt/wechat/`，启动器 `/usr/bin/wechat` 是指向 `/opt/wechat/wechat` 的软链
  （软链由 postinst 建，postrm 删）
- 依赖 `fonts-noto-cjk | google-noto-cjk-fonts`
- 桌面项 `/usr/share/applications/wechat.desktop`，图标在 `/usr/share/icons/hicolor/512x512/apps/wechat.png`
- 安装体积约 770 MB

## 安装方式清单

**装的时候选了哪种，以后更新就用哪种。** 换了方式会出现两份微信同时在 PATH / 应用菜单里。

| 安装方式 | 识别特征 | 更新命令 |
| --- | --- | --- |
| Flatpak | `flatpak list --app` 里有 `com.tencent.WeChat` | `flatpak update com.tencent.WeChat` |
| deb（apt / dpkg） | `dpkg -l` 里有 `wechat`；`/usr/bin/wechat` 存在 | 重新下载 deb 后 `sudo apt install ./WeChatLinux_x86_64.deb` |
| rpm（dnf / yum） | `rpm -q wechat` 有输出；`/usr/bin/wechat` 存在 | 重新下载 rpm 后 `sudo dnf install ./WeChatLinux_x86_64.rpm` |
| AppImage | 家里有 `WeChatLinux_*.AppImage` 文件 | 下载新 AppImage 覆盖旧文件，重新 `chmod +x` |
| AUR（Arch 系） | `pacman -Q` 里有 `wechat-bin` / `wechat-appimage` / `tencent-wechat` | `paru -S wechat-bin`（沿用当初的包名） |

### Flatpak（推荐）

社区打包（flathub/com.tencent.WeChat），内容就是官方 deb：

```bash
flatpak install flathub com.tencent.WeChat
flatpak run com.tencent.WeChat
```

沙箱隔离，卸载干净（`flatpak uninstall com.tencent.WeChat`），不往系统里塞 `/opt/wechat`。
缺点：文件选择、截图、通知等需要额外授权，第一次跑会弹权限对话框。

### deb

```bash
curl -LO https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.deb
sudo apt install ./WeChatLinux_x86_64.deb
```

`apt install` 会自动补 `fonts-noto-cjk` 等依赖，比 `dpkg -i` 省事；用 `dpkg -i` 的话装完记得 `sudo apt -f install` 修依赖。

### rpm

```bash
curl -LO https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.rpm
sudo dnf install ./WeChatLinux_x86_64.rpm
```

### AppImage

```bash
curl -LO https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.AppImage
chmod +x WeChatLinux_x86_64.AppImage
./WeChatLinux_x86_64.AppImage
```

需要 fuse2；AppImage 自带更新的话可能因为腾讯的下载域名策略失败，失败就手动下载覆盖。

### AUR（Arch 系）

| 包名 | 说明 |
| --- | --- |
| `wechat-bin` | 官方 deb 的重打包，日常用这个 |
| `wechat-appimage` | 官方 AppImage 的重打包 |
| `tencent-wechat` | 社区重打包，主打原生 Wayland 支持和隐私沙箱 |
| `wechat-universal-bwrap` | 带 bwrap 沙箱的版本 |
| `wechat` | Kraftland 的 portable 版（沙箱支持与增强） |

```bash
paru -S wechat-bin
```

## 检测当前安装方式

更新前先检测，再沿用同一种方式：

```bash
# 1. Flatpak
flatpak list --app | grep -i wechat

# 2. deb / rpm（系统包）
dpkg -l | grep -w wechat          # deb 系
rpm -q wechat                     # rpm 系

# 3. AUR
pacman -Q | grep -i wechat

# 4. AppImage / 手工放置
ls ~/Applications/WeChatLinux*.AppImage 2>/dev/null
ls ~/.local/bin/WeChatLinux* 2>/dev/null
```

看真实路径确认：

```bash
readlink -f "$(command -v wechat)"   # deb/rpm 会指向 /opt/wechat/wechat
```

判断不出来就问用户，不要猜，更不要擅自换方式重装。

## 更新

### Flatpak

```bash
flatpak update com.tencent.WeChat
```

### deb / rpm

官方没有 apt 源，只能手动下载新的 deb/rpm 再装：

```bash
# 升级前先退出正在运行的微信，否则数据文件被占用
sudo apt install ./WeChatLinux_x86_64.deb
```

deb 文件名不带版本号，重新下到的就是最新版；`apt` 会比较版本号，只有版本变大才真正升级。
确认版本：

```bash
dpkg -l wechat        # 或 rpm -q wechat
```

### AppImage

```bash
curl -LO https://dldir1v6.qq.com/weixin/Universal/Linux/WeChatLinux_x86_64.AppImage
chmod +x WeChatLinux_x86_64.AppImage
```

覆盖前先退出会运行的旧实例。

### AUR

```bash
paru -S wechat-bin   # 用当初装的那个包名
```

## 从 deb 版迁移到 Flatpak

换安装方式时，**聊天记录不会自动跟着走**，要手动搬（实测 2026-11 做过一次，deb 4.1.1.4 → flatpak 4.1.13.26）。

数据目录在两边的位置不一样：

| 内容 | deb 版 | Flatpak 版 |
| --- | --- | --- |
| 聊天记录与文件 | `~/文档/xwechat_files`（或 `~/Documents/xweChat_files`） | `~/.var/app/com.tencent.WeChat/xwechat_files` |
| 配置、登录态、日志 | `~/.xwechat` | `~/.var/app/com.tencent.WeChat/.xwechat` |

flatpak 版的这两个目录由 manifest 的 `persistent=.xwechat;xwechat_files` 声明，落在 app 目录下。

步骤：

```bash
# 1. 先退出正在运行的微信（否则文件被占用）
pkill -TERM -f '/usr/bin/wechat'

# 2. 卸载 deb 版
sudo apt remove wechat
sudo apt purge wechat      # 清掉 dpkg 里残留的 rc 状态

# 3. 装 flatpak 版
flatpak install -y flathub com.tencent.WeChat

# 4. 先启动一次，让它生成目录结构，然后退出
flatpak run com.tencent.WeChat   # 跑起来，等窗口出来，退出

# 5. 搬数据（先跑一次 flatpak 生成目录，再拷，不然 cp 会报错）
cp -a ~/文档/xwechat_files/. ~/.var/app/com.tencent.WeChat/xwechat_files/
cp -a ~/.xwechat/.              ~/.var/app/com.tencent.WeChat/.xwechat/
rm -f ~/.var/app/com.tencent.WeChat/.xwechat/lock/lock.ini   # 清旧锁

# 6. 再启动，扫码登录
flatpak run com.tencent.WeChat
```

搬完确认旧账号目录还在（`wxid_xxx`），说明记录没丢：

```bash
ls ~/.var/app/com.tencent.WeChat/xwechat_files/
```

### 坑

- **必须先退出微信再搬**。微信运行时会持有 sqlite 和 `lock/lock.ini`，硬拷会拷到写了一半的文件。
- **`~/文档` 还是 `~/Documents` 取决于系统语言**，两个路径都看一眼再拷，别只拷一个。
- 旧版目录名是 `xweChat_files`，新版微信会提示改名成 `WeChat Files`；按提示改，别自己删。
- flatpak 版启动日志里的 `Fontconfig warning: ... cache files were generated by a newer version`
  是宿主机 fontconfig 比沙箱内旧，不影响使用，可以不管。
- 迁移后旧目录还在（`~/文档/xwechat_files`、`~/.xwechat`，本例 1.9 GB），
  确认新版本登录后聊天记录完整再删。

## 第三方方案现状

新装别用，仅供了解。

| 方案 | 状态 |
| --- | --- |
| `Huang-Hao/wechat-uos`（UOS 版 Electron 封装，AUR `electronic-wechat-uos-bin`） | 2022 年后停更，腾讯收紧 Web 端协议后普遍登录失败（`user kv not init`） |
| Wine 打包版（`jcleng/WineWechat-linux-flatpak`、AUR `mejituu-wine8`） | 能跑但字体、闪退、多开都是坑，且仍在风控检测名单里 |
| 协议类机器人（`wechaty` / `itchat-uos`） | 只能收发消息，当不了正常客户端；协议收紧后多数已失效 |
| `lovechoudoufu/wechat_for_linux` | 只是搬运官方 / 统信安装包，已归档 |

## 注意

- **升级前先退出微信**，运行中直接装 deb 会因为文件被占用而失败，或导致聊天记录写坏。
- 聊天记录与文件在 `~/Documents/WeChat Files`（旧版本目录名是 `~/Documents/xweChat_files`）。
  换目录名时按微信启动时的提示重命名，别直接删。
- **Wayland 下**窗口不能置顶、无法全局快捷键唤出、输入法候选框位置常偏。用 XWayland 跑最省事：
  `wechat --force-x11` 或 `XDG_CURRENT_DESKTOP=... QT_QPA_PLATFORM=xcb wechat`。
  Arch 上可装 `wechat-xwayland`（KDE 激活命令）、`plasma-wechat-toggle`、`wechat-callout`。
- 中文显示依赖 Noto CJK；`dpkg -l wechat` 里 `fonts-noto-cjk` 缺失会导致方块字。
- 官方 Linux 版登录偶尔要求扫码确认，别急着反复登录，等几十秒再试。
- 用 root 跑会崩（Chromium 沙箱限制），改用普通用户；老的 UOS 包装需要 `--no-sandbox`，
  官方新版本不需要，别照抄老教程。
- 腾讯的下载域名是 `dldir1v6.qq.com`，直链经常被 CDN 换 IP；`curl -LO` 失败时重试一次即可。
- 换安装方式（deb ↔ Flatpak ↔ AppImage）时聊天记录不会自动迁移，要手动搬，
  见上面「从 deb 版迁移到 Flatpak」。

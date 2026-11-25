# Linksy

- 桌面端仓库：https://github.com/LinkSy-mg/linksy-desktop
- 分发站（Android 安装包 / 各平台安装包）：https://github.com/LinkSy-mg/linksy-releases
- 官网：https://linksy.diary.mg

把安卓手机当成电脑的外设：通过 Wi-Fi 或 USB 互传文件与剪贴板、查看/回复手机通知、
把手机当第二显示器 / 摄像头 / 麦克风、在手机上看电脑屏幕并接管输入、用电脑终端操作手机。
无账号、无云端、无服务器，局域网内直连。

桌面端分 GTK 版（GNOME 等）和 Qt 版（KDE Plasma），加一个后台常驻服务 `linksyd`。

## 资产与安装（按平台）

### Linux

桌面端 Release（`LinkSy-mg/linksy-desktop`）的资产：

| 资产 | 用途 |
| --- | --- |
| `linksy-<版本>.flatpak` | Flatpak 单文件包，自带 GTK 应用，**无需 root** |
| `linksy-<版本>-linux-x86_64.tar.gz` | 免安装 tarball，解压即用 |
| `linksy-common_<版本>_amd64.deb` / `linksy-<版本>-1.x86_64.rpm` | 公共依赖包 |
| `linksy_<版本>_amd64.deb` / `linksy-<版本>-1.x86_64.rpm` | GTK 版应用 |
| `linksy-qt_<版本>_amd64.deb` / `linksy-qt-<版本>-1.x86_64.rpm` | Qt 版应用（Plasma） |

另有 `.suse.x86_64.rpm` 变体，是给 openSUSE 打的，Fedora/Debian 不要选。

桌面端 Flatpak 不在 Flathub 上，只能从 Release 下 `.flatpak` 单文件包装。用户级安装（推荐，不需要 sudo）：

```bash
mkdir -p tmp/linksy
gh release download <tag> --repo LinkSy-mg/linksy-desktop \
  --pattern 'linksy-<版本>.flatpak' --pattern 'SHA256SUMS' --dir tmp/linksy --clobber
cd tmp/linksy && sha256sum -c --ignore-missing SHA256SUMS
flatpak install --user --bundle ./linksy-<版本>.flatpak
```

系统级安装用 `sudo dnf install ./linksy-<版本>-1.x86_64.rpm`（要连 `linksy-common` 一起装），
Debian 系用 `sudo apt install ./linksy-common_<版本>_amd64.deb ./linksy_<版本>_amd64.deb`。

### Android

手机端在 `LinkSy-mg/linksy-releases`（`linux-v…` 之外的 Android Release），
资产是 `linksy-<版本>-apk`（arm64）/ `linksy-<版本>-armv7.apk` / `linksy-<版本>.apk`（通用）。
装好后在电脑和手机两边打开 Linksy，两边显示同一个配对码即可连上。

## 命令行入口

Flatpak 里的命令不直接暴露在 PATH 上，包了一个包装脚本到 `~/.local/bin/linksy`：

```sh
#!/bin/sh
exec flatpak run --command=linksy mg.diary.linksy.Linksy "$@"
```

用法见 `linksy --help`：`devices` / `pair` / `invite` / `history` / `extend` / `camera` /
`mic` / `screen` / `send` / `notify` / `clip` / `open` 等。

用 tarball 或 rpm/deb 装的话，命令就叫 `linksy`（rpm/deb 在 `/usr/bin`，tarball 在 `~/.local/bin`）。

## 防火墙

需要 UDP 47200（Linksy 协议）和 UDP 5353（mDNS）。Fedora / openSUSE 用 firewalld：

```bash
sudo firewall-cmd --permanent --add-port=47200/udp
sudo firewall-cmd --permanent --add-service=mdns
sudo firewall-cmd --reload
```

应用的「Finish Setup」引导页也能一键配好，不用手动敲。

## 更新

Flatpak 单文件包没有升级通道，更新 = 重新下载新版本 `.flatpak` 再装一次：

```bash
gh release download <新tag> --repo LinkSy-mg/linksy-desktop \
  --pattern 'linksy-<新版本>.flatpak' --dir tmp/linksy --clobber
flatpak install --user --bundle --noninteractive ./tmp/linksy/linksy-<新版本>.flatpak
```

rpm/deb 装的就用 `sudo dnf install ./linksy-<新版本>-1.x86_64.rpm` 覆盖安装。

上游还提供一键脚本 `curl -fsSL https://linksy.diary.mg/install.sh | sh`，
它会自己探测系统并选择包管理器；用之前先看清它要做什么。

## 注意

- **`.flatpak` 包装运行时优先选 `org.gnome.Platform//51`**，只有 flathub 提供。
- **`flatpak install --bundle` 提示「将其配置为新的远程仓库 flathub-1」**：
  本机 user 和 system 两个安装都叫 `flathub`，flatpak 解析运行时来源时出现歧义，
  于是想新建一个指向 `dl.flathub.org` 的远程仓库。直接选 `n` 取消，
  再用 `flatpak install --user --bundle --noninteractive <包>` 安装即可——
  运行时在已有的 user remote（国内 USTC 镜像）里已经有了，不会新建远程仓库。
- **`flatpak run` 不响应 SIGTERM**：`timeout 20 flatpak run ...` 不会按时返回，
  命令会一直挂着直到手动 `pkill -f linksy-app`。测完记得清理后台进程。
- **Flatpak 版有宿主机访问权限**（`org.freedesktop.Flatpak`），所以开机自启、摄像头驱动、
  USB 连接和手机终端（用的是宿主机自己的 shell）这些功能都能用。
- **手机终端功能依赖宿主机 shell**，flatpak 沙箱默认只放开了下载/图片等目录，
  真要用「终端」功能时留意应用内 Finish Setup 的授权提示。
- **只从上述仓库或官网下载**，别处下的副本可能被改过。

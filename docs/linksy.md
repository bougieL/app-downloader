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

## 能不能像苹果那样在电脑上直接操作手机

分两种情况：

- **USB 线连接时可以**：桌面端 0.11 起，手机插着数据线时，电脑端通过 Android Open Accessory（AOA 2.0）
  给手机接一个虚拟触摸屏（外部 `SOURCE_TOUCHSCREEN`），电脑上的点击和键盘就转发进手机。
  手机端不需要开发者选项，也不需要无障碍权限。手机会显示「Controlled from <computer>」。
- **走 Wi-Fi 时不行**：往手机注入触摸需要 `AccessibilityService`，而 Android 的反欺诈机制会拦截
  侧载 APK 声明这类权限的应用（上游 v0.1.3/0.1.4 就是因此发不出去），所以手机端直接回
  `decline_unsupported`，上游文档写明「不打算在没有线缆的情况下支持」。
  Wi-Fi 下只能**看**手机屏幕（手机共享给电脑），不能点。

  这不是协议限制（协议里 `control` 会话早就定义好，两端代码都跑得通），
  也不是技术上做不到，而是 Android 的权限模型挡着：

  | 路线 | 为什么 Linksy 不走 |
  | --- | --- |
  | 无障碍服务 `AccessibilityService` | 侧载 APK 声明无障碍会被安装拦截，且 Android 13+ 还要手动「允许受限设置」；上游 ROADMAP 把它列进「Not planned」，issue #8 已被维护者关闭 |
  | root + uinput | 原理和线缆上的 AOA 虚拟触摸屏一样，但要 root |
  | ADB over Wi-Fi | scrcpy 的路子，要开开发者选项 + USB 调试 + `adb tcpip` |

  反方向（手机当触控板操作**电脑**的屏幕）Wi-Fi 下是可以的，桌面端需要 RemoteDesktop portal，GNOME 有。

  线缆方案也不是所有机型都灵：上游说「works on some phones」，
  适配范围在 issue #5 里跟踪。

Wi-Fi 下仍然能做的（相当于苹果连续互通的一部分）：手机通知/短信/邮件汇总到电脑的 Inbox，
部分应用支持在电脑里直接回复；文件双向传、剪贴板电脑→手机；响铃和查找手机；媒体播放控制；
手机当摄像头、麦克风、第二显示器；在手机上用电脑的终端。

Wi-Fi 下往手机**打字**也只有土办法：Send Clipboard 整段粘贴，不支持实时输入、退格和快捷键。
上游 issue #107 提议做一个「Linksy Keyboard」输入法（IME 不需要无障碍权限），
维护者暂时没接受但把 issue 留着了。

用线缆控制时注意两点：

- Android 15 的屏幕共享保护会让状态栏和通知栏在画面里全黑，任何应用都绕不过去，
  需要在手机上开「开发者选项 › 关闭屏幕共享保护」后重新共享。
- 手机共享屏幕始终是整屏（Android 14+ 否则默认预选「单个应用」，黑边）。

电脑端的桌面环境不影响用线控制：触摸事件是在 Linksy 自己的窗口里捕获再转发的，
不需要系统级的虚拟输入设备，也不需要 RemoteDesktop portal，GNOME/Wayland 下同样可用。

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

# GNOME Tweaks（微调）

- 命令：`gnome-tweaks`
- 桌面项：`org.gnome.tweaks.desktop`
- 运行环境：Fedora 45 (Workstation) / x86_64 / GNOME Shell 51（Wayland）

改 GNOME 设置项的图形工具（外观、字体、键盘快捷键、顶部栏、窗口标题栏按钮等）。
它改的是 dconf/gsettings，**和 Just Perfection 扩展有部分功能重叠**，
两者调同一个键时以后写入的为准。

## 安装

Fedora 官方包，走 dnf（**需要 sudo**）：

```bash
sudo dnf install -y gnome-tweaks
```

装完桌面项就有了（应用列表里叫「Tweaks」/「微调」）。

版本跟着 GNOME 走：Fedora 45 的仓库里是 **49.0**（`gnome-tweaks-49.0-4.fc45.noarch`）。
版本号不等于系统的 GNOME 版本（GNOME 51 配 Tweaks 49.0 是正常的），
只要 dnf 能装上就是 Fedora 官方适配过的版本。

## 更新

```bash
sudo dnf upgrade gnome-tweaks
```

## 常用设置项

| 位置 | 作用 |
| --- | --- |
| 外观 → 字体 | 改系统 UI 字体（**不是终端字体**） |
| 外观 → 顶栏 | 顶栏高度、位置（顶/底）、时钟显示方式 |
| 外观 → 标题栏 | 标题栏按钮位置（左右）、最小化/最大化/关闭的排列 |
| 键盘和鼠标 | 快捷键（Super+数字切工作区等）、鼠标加速度 |

改完**即时生效**，不用注销，也不用重启 shell。

## 注意

- **Tweaks 不是 GNOME 扩展**，装的是系统包，`gnome-extensions list` 里不会有它。
  扩展相关的设置（顶栏隐藏、dock 行为）要用 Just Perfection，
  见 [gnome-extensions.md](gnome-extensions.md)。
- Tweaks 只改 dconf，**改不了 fontconfig**（`monospace` / `sans-serif` 的解析规则）。
  中文回退那套要改 `~/.config/fontconfig/fonts.conf`，见 [fonts.md](fonts.md)。
- **和 Just Perfection 撞键**：两边的「顶栏」「概览」开关写的是同一批 gsettings，
  只留一个在改就行，不然改一个另一个立刻改回去。
- 字体设置项改的是 `org.gnome.desktop.interface font-name`，
  只影响 GTK 应用；GNOME Shell 自己（顶栏、概览）走主题字体。
- 本机状态（2026-11-21）：Fedora 官方包 49.0，`/usr/bin/gnome-tweaks`。

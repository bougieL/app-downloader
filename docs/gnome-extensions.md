# GNOME 扩展

- 扩展仓库：https://extensions.gnome.org
- 用户扩展目录：`~/.local/share/gnome-shell/extensions/<uuid>/`
- 启用列表：dconf 键 `/org/gnome/shell/enabled-extensions`（`gsettings get org.gnome.shell enabled-extensions`）
- 运行环境：Fedora 45 (Workstation) / x86_64 / GNOME Shell 51（Wayland）

GNOME 扩展不走 GitHub Release，分发站是 extensions.gnome.org（下称 EGO），装在用户目录，不需要 sudo。

## 安装方式清单

| 方式 | 识别特征 | 更新方式 |
| --- | --- | --- |
| A. shell 内置安装（D-Bus `InstallRemoteExtension`） | 弹窗确认后由 shell 自己下载解压，扩展随即加载 | 同一 uuid 再调一次 D-Bus 方法 |
| B. EGO zip 手工安装 | 目录里的文件是手工解压进去的 | 重新下载 zip 覆盖，再重新登录 |
| C. Flatpak Extension Manager（`com.mattjakeman.ExtensionManager`） | `flatpak list` 里有它 | `flatpak update com.mattjakeman.ExtensionManager` |
| D. 系统包 | `rpm -q gnome-shell-extension-*` 有对应包 | `sudo dnf upgrade` |

## 扩展管理器（Extension Manager）

图形化的扩展管理器，官方仓库 `com.mattjakeman.ExtensionManager`（Flathub），
可以搜索、安装、启停、配置 EGO 上的扩展，还能自己装本地 zip。

```bash
flatpak remote-add --user --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install --user -y flathub com.mattjakeman.ExtensionManager
flatpak run com.mattjakeman.ExtensionManager
```

- 装用户级（`--user`）不用 sudo、不用 polkit 授权。系统已有的 flathub remote 是 system 级，
  `--user` 装之前得先把 flathub 加成用户 remote，否则会报「未发现用于 "flathub" 的远程引用」。
- 会顺带拉 `org.gnome.Platform/x86_64/51` 运行时，几百 MB，耐心等。
- 装上之后 GNOME Shell 才会做扩展更新检查：shell 的 `updatesSupported` 要求系统里存在
  `com.mattjakeman.ExtensionManager.desktop` 或 `org.gnome.Extensions.desktop`，两者都没有就完全不检查更新。
- 桌面入口在 `~/.local/share/flatpak/exports/share/applications/com.mattjakeman.ExtensionManager.desktop`。

## 检测当前安装方式

```bash
gnome-extensions list                            # shell 已加载的扩展
ls ~/.local/share/gnome-shell/extensions/         # 用户目录里有哪些扩展
flatpak list | grep -i extension                  # 有没有用 Flatpak 版扩展管理器
rpm -qa | grep gnome-shell-extension             # 有没有用系统包装的扩展
```

看 `~/.local/share/gnome-shell/extensions/<uuid>/metadata.json` 的 `version` 字段，
和 EGO 页面上该扩展的最新版对比，可以判断是不是最新。

## 安装（EGO 直装，无需 sudo）

```bash
uuid="dash-to-dock@micxgx.gmail.com"
shell_ver=$(gnome-shell --version | awk '{print $NF}' | cut -d. -f1)   # 51

mkdir -p ~/.local/share/gnome-shell/extensions/${uuid}
curl -fL "https://extensions.gnome.org/download-extension/${uuid}.shell-extension.zip?shell_version=${shell_ver}" \
  -o /tmp/${uuid}.zip
unzip -oq /tmp/${uuid}.zip -d ~/.local/share/gnome-shell/extensions/${uuid}

# 启用：把 uuid 加进 enabled-extensions（已存在的不重复加）
current=$(gsettings get org.gnome.shell enabled-extensions | tr -d "[]" | tr ',' '\n' | tr -d "' " | awk 'NF')
new=$(printf '%s\n%s\n' "$current" "$uuid" | awk 'NF && !seen[$0]++' | sed "s/.*/'&'/" | paste -sd, -)
gsettings set org.gnome.shell enabled-extensions "[$new]"
```

装完核对 `metadata.json` 里的 `shell-version` 是否包含当前 GNOME 版本（`51`），
EGO 上"有匹配版本"不代表 metadata 一定声明支持。

## 立即生效（不想重新登录）

GNOME Shell 只在启动时扫描扩展目录，运行期没有目录监听，所以手工放进去的扩展要重新登录才加载。
想立刻生效，让 shell 自己装（会弹窗确认，点 Install）：

```bash
gdbus call --session --dest org.gnome.Shell.Extensions \
  --object-path /org/gnome/Shell/Extensions \
  --method org.gnome.Shell.Extensions.InstallRemoteExtension "<uuid>"
```

这个方法自带下载、解压、加载、启用，一步到位，适合批量安装时逐个点确认。

## EGO 接口

| 用途 | 地址 |
| --- | --- |
| 查扩展（列表/详情） | `https://extensions.gnome.org/api/v1/extensions/?uuid=<uuid>` |
| 查元信息 | `https://extensions.gnome.org/extension-info/?uuid=<uuid>&shell_version=<版本>` |
| 下载（自动匹配 shell 版本） | `https://extensions.gnome.org/download-extension/<uuid>.shell-extension.zip?shell_version=<版本>` |
| 下载（指定版本） | 同上，改用 `?version_tag=<pk>` |
| 扩展详情页 | `https://extensions.gnome.org/extension/<id>/<slug>/`，页面里的 `data-versions` 属性给出每个 shell 版本对应的 pk |

## 更新

沿用当初的安装方式：

- 方式 A：重新调一次 D-Bus `InstallRemoteExtension`，shell 会下载最新版覆盖。
- 方式 B：重跑上面的下载 + 解压命令覆盖，再重新登录（或用方式 A 立即生效）。
- 方式 C：扩展管理器（Extension Manager）里点更新；管理器本身用
  `flatpak update com.mattjakeman.ExtensionManager` 更新。
- 方式 D：交给各自的包管理器。

扩展管理器装好后，GNOME Shell 的扩展更新检查才会启用（见上面「扩展管理器」一节），
扩展有新版时 shell 会在扩展菜单里提示。

更新前先看 `metadata.json` 里的 `version` 字段，别盲目覆盖正在用的版本。

## 卸载

用户目录（方式 A/B/C）的扩展直接删目录：

```bash
rm -rf ~/.local/share/gnome-shell/extensions/<uuid>
```

系统包（方式 D）要删对应的 RPM 包，删完包目录 `/usr/share/gnome-shell/extensions/<uuid>/` 才会消失：

```bash
rpm -qf /usr/share/gnome-shell/extensions/<uuid>/   # 先查是哪个包
sudo dnf remove gnome-shell-extension-<name>
```

系统包装的扩展还留在 `org.gnome.shell enabled-extensions` 里的话，顺手清掉，
否则 shell 每次启动都去加载一个不存在的 uuid：

```bash
current=$(gsettings get org.gnome.shell enabled-extensions | tr -d "[]" | tr ',' '\n' | tr -d "' " | awk 'NF')
new=$(printf '%s\n' "$current" | grep -vx '<uuid>' | awk '!seen[$0]++' | sed "s/.*/'&'/" | paste -sd, -)
gsettings set org.gnome.shell enabled-extensions "[$new]"
```

删包之后**必须注销重登**（Wayland 不能原地重启 shell），
`gnome-extensions list` 读的是运行中 shell 的内存状态，重登前仍会列出已删的扩展。

## 注意

- **`gnome-extensions install <uuid>` 在 GNOME 51 只接受本地 zip 文件**，不支持直接用 UUID 从 EGO 拉：

  ```text
  $ gnome-extensions install dash-to-dock@micxgx.gmail.com
  打开文件 dash-to-dock@micxgx.gmail.com 时出错：没有那个文件或目录
  ```

  它把参数当本地路径处理了。要从 EGO 装就用上面的 `gdbus` 方法，或自己下载 zip 再 `gnome-extensions install <zip>`。

- **`gnome-extensions enable <uuid>` 对刚装上还没被 shell 扫描到的扩展报「扩展不存在」**。
  原因同上：shell 的 `js/ui/extensionSystem.js` 里 `_loadExtensions()` 只在启动时执行一次，
  没有对扩展目录做 inotify 监听。现象是命令返回「扩展"xxx"不存在」，
  但 `gsettings get org.gnome.shell enabled-extensions` 里其实已经写进去了，重新登录后照常生效。

- **`extension-info` 接口两个参数都得带**，少一个就 404：

  ```bash
  # 404
  curl -s "https://extensions.gnome.org/extension-info/?uuid=dash-to-dock@micxgx.gmail.com"
  # 正常返回 JSON
  curl -s "https://extensions.gnome.org/extension-info/?uuid=dash-to-dock@micxgx.gmail.com&shell_version=51"
  ```

- **uuid 会变**，旧 uuid 会 404。例如托盘图标扩展现在是
  `appindicatorsupport@rgcjonas.gmail.com`，老的 `appindicatorsupport@martinottas`
  在 EGO 上已经没有记录。装之前先查 EGO 确认 uuid：

  ```bash
  curl -s "https://extensions.gnome.org/api/v1/extensions/?uuid=<uuid>" | head -c 200
  ```

  返回 `{"count":0,...}` 就是没有。

- **Wayland 下不能原地重启 GNOME Shell**（`Alt+F2` → `r` 只对 X11 有效），
  新装的扩展只能注销重登，或者走 D-Bus 安装通道立即生效。

- **`/api/v1/extensions/<uuid>/` 详情接口 404**，只有列表接口
  `/api/v1/extensions/?uuid=<uuid>` 可用；要拿版本信息得解析详情页里的 `data-versions`。

- **zip 里是平铺的文件**（`metadata.json`、`extension.js` 直接在根目录），
  解压到 `~/.local/share/gnome-shell/extensions/<uuid>/` 即可，不要多套一层目录。

- **删 `gnome-shell-extension-*` 包会连带删掉 `gnome-classic-session`**：

  ```text
  $ sudo dnf remove gnome-shell-extension-apps-menu
  正在移除依赖软件包:
   gnome-classic-session
  ```

  `gnome-classic-session` 依赖 apps-menu / places-menu / window-list / launch-new-instance
  这几个扩展，删了它们，Classic 会话就没了，登录界面里的「GNOME Classic」选项也会消失。
  用 `--setopt=clean_requirements_on_remove=False` 也拦不住（硬依赖照样删）。
  确实还要用 Classic 就得连那几个扩展一起留着，或者之后重装 `gnome-classic-session`。

- **本机状态（2026-11-19 更新）**：Fedora 45 (Workstation) / x86_64 / GNOME Shell 51，
  扩展装在 `~/.local/share/gnome-shell/extensions/`，共 11 个：

  | 扩展 | uuid |
  | --- | --- |
  | Dash to Dock | `dash-to-dock@micxgx.gmail.com` |
  | AppIndicator and KStatusNotifierItem Support | `appindicatorsupport@rgcjonas.gmail.com` |
  | Just Perfection | `just-perfection-desktop@just-perfection` |
  | User Themes | `user-theme@gnome-shell-extensions.gcampax.github.com` |
  | Blur my Shell | `blur-my-shell@aunetx` |
  | Caffeine | `caffeine@patapon.info` |
  | Clipboard Indicator | `clipboard-indicator@tudmotu.com` |
  | Vitals | `Vitals@CoreCoding.com` |
  | GSConnect | `gsconnect@andyholmes.github.io` |
  | Tiling Assistant | `tiling-assistant@leleat-on-github` |
  | Rounded Window Corners | `rounded-window-corners@fxgn` |

  扩展管理器：`com.mattjakeman.ExtensionManager` 0.7.0，Flatpak 用户级安装
  （`flatpak install --user`），运行时 `org.gnome.Platform/x86_64/51`。

  `enabled-extensions` 里还留着两条没装对应的残留 uuid：
  `gtk4-ding@smedius.gitlab.com`（本机既没 RPM 包也没用户目录）和
  `Rounded_Corners@lennart-k`（旧 uuid，已被 `rounded-window-corners@fxgn` 取代）。

  2026-11-19 卸掉了 5 个 Fedora 自带扩展（系统包，方式 D）：
  apps-menu、background-logo、launch-new-instance、places-menu、window-list，
  同时连带删掉了 `gnome-classic-session`。

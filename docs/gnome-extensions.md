# GNOME 扩展

- 扩展仓库：https://extensions.gnome.org
- 用户扩展目录：`~/.local/share/gnome-shell/extensions/<uuid>/`
- 启用列表：dconf 键 `/org/gnome/shell/enabled-extensions`（`gsettings get org.gnome.shell enabled-extensions`）
- 运行环境：Fedora 45 (Workstation) / x86_64 / GNOME Shell 51（Wayland）
- 换主题另见 [gnome-themes.md](gnome-themes.md)（结论：本机不换）

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
  本机已经有用户级 flathub remote（`flatpak remotes --user` 里能看到 `flathub` 和 `flathub-1`），
  直接装即可。
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

# 必须编译 GSettings schema，否则扩展加载失败（EGO 的 zip 不带编译好的 gschemas.compiled）
[ -d ~/.local/share/gnome-shell/extensions/${uuid}/schemas ] && \
  glib-compile-schemas ~/.local/share/gnome-shell/extensions/${uuid}/schemas

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

- **`extension-info` 的 JSON 里没有可用的 shell 版本字段**（`shell_version` 取到 `null`），
  判断「支不支持当前 GNOME 版本」要等 zip 解压后看
  `~/.local/share/gnome-shell/extensions/<uuid>/metadata.json` 的 `shell-version` 数组：

  ```bash
  python3 -c "import json;print(json.load(open('metadata.json'))['shell-version'])"
  ```

  `download-extension/<uuid>.shell-extension.zip?shell_version=51` 返回 200 不代表
  该扩展声明支持 51——EGO 会给最近的一个可用版本。

- **`api/v1/extensions/?uuid=<uuid>` 只能判存在性**（`count` 字段），
  拿不到版本号；版本号看 `metadata.json` 的 `version`，或看 EGO 详情页的 `data-versions`。

- **Extension Manager 会「悄悄关掉」第三方扩展**：它检查扩展声明的 `shell-version`，
  不支持当前 GNOME 版本的就禁用掉（GNOME 自带的扩展管理器也是这个行为，
  换主题文档里提到的「第三方扩展被悄悄关掉」就是这个来源）。
  本机装的 10 个都声明支持 51，正常不会被禁。

- **手工装的扩展必须自己编译 GSettings schema**：EGO 的 zip 里只有 `.gschema.xml`，
  **没有编译好的 `gschemas.compiled`**（少数扩展例外，比如 AppIndicator 自带），
  而 GNOME Shell 不会替扩展编译，直接用
  `Gio.SettingsSchemaSource.new_from_directory()` 读那个文件，读不到就整个扩展加载失败：

  ```text
  Extension caffeine@patapon.info: GLib.FileError: 打开文件
  “.../caffeine@patapon.info/schemas/gschemas.compiled”失败：没有那个文件或目录
  ```

  装完挨个编译：

  ```bash
  for d in ~/.local/share/gnome-shell/extensions/*/schemas/; do
    [ -f "$d/gschemas.compiled" ] || glib-compile-schemas "$d"
  done
  ```

  Extension Manager（Flatpak）装的时候会自动做这一步，所以走它装的一般不会踩。

- **加载失败的扩展会「卡死」在 ERROR 状态，reload 救不回来**：GNOME Shell 的
  `extensionSystem.js` 里 `enableExtension()` 只在状态是 `DISABLED` 时才重新 init，
  ERROR 状态直接跳过；`disableExtension()` 也只把 ACTIVE 的置为 DISABLED，
  **ERROR 的原样留着**。所以编译完 schema 后：

  - `gnome-extensions disable/enable <uuid>`：无效，状态还是 ERROR；
  - `gnome-extensions info <uuid>`：只显示「状态: ERROR」，**不给错误原因**；
  - `org.gnome.Shell.Extensions.ReloadExtension`：**接口在 D-Bus 上有，实现是空的**，
    调用报 `Method ReloadExtension is not implemented`；
  - `org.gnome.Shell.Extensions.GetExtensionErrors <uuid>`：能拿到真实报错文本
    （`GLib.FileError: ... gschemas.compiled ...`）；
  - `gnome-shell --replace`：Wayland 下这个版本**不支持**（`未知选项 --replace`）。

  真正能救的只有两条路：**注销重登**（新 shell 会重新扫描并加载），
  或者 `UninstallExtension` + `InstallRemoteExtension`（后者会弹确认框，要人点）。

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

  本机 2026-11-21 就是这么把这 5 个 Fedora 自带扩展一次性删掉的，
  `gnome-classic-session` 和 `gnome-shell-extension-common` 被连带移除，
  现在 `/usr/share/gnome-shell/extensions/` 是空的。
  删包后**要注销重登**才真正消失（Wayland 下不能原地重启 shell）；
  重登前 `gnome-extensions list` 仍会列出已删的扩展（读的是运行中 shell 的内存状态）。

- **Dock 图标上的角标（未读数）没有红底了**，只剩一个白色的数字：

  红色圆角底色是 **Dash to Dock 自己扩展的 `stylesheet.css`** 画的，
  不是 GNOME Shell 主题画的（shell 自带的 `gnome-shell-{dark,light,high-contrast}.css`
  里根本没有 `.notification-badge` 这条规则，shell 主题 CSS 打包在
  `/usr/share/gnome-shell/gnome-shell-theme.gresource` 里，用 `gresource`/`gjs` 拆出来才能查）：

  ```css
  #dashtodockContainer .notification-badge,
  .overview-tile .notification-badge,
  .icon-grid .notification-badge {
    color: white;
    background-color: red;
    ...
  }
  ```

  本机的原因：扩展目录里的 `stylesheet.css` 被改名成了 `stylesheet.css.bak`，
  扩展基类只在存在 `stylesheet.css` 时才加载样式表，于是整份 d2d 样式都没生效
  （角标数字还在，因为它只是文本；红底没了，因为那是 CSS 给的）。
  查法：

  ```bash
  ls ~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com/stylesheet.css*
  ```

  修法：把文件改回原名，再让扩展重新加载（Wayland 下不用重登）：

  ```bash
  cd ~/.local/share/gnome-shell/extensions/dash-to-dock@micxgx.gmail.com
  cp stylesheet.css.bak stylesheet.css
  gnome-extensions disable dash-to-dock@micxgx.gmail.com
  gnome-extensions enable  dash-to-dock@micxgx.gmail.com
  ```

  同理，任何扩展目录里被改名的 `stylesheet.css` 都会让该扩展「能用但没样式」。

- **本机状态（2026-11-21）**：Fedora 45 (Workstation) / x86_64 / GNOME Shell 51，
  扩展装在 `~/.local/share/gnome-shell/extensions/`（用户目录，方式 B），共 10 个：

  | 扩展 | uuid | 版本 |
  | --- | --- | --- |
  | Dash to Dock | `dash-to-dock@micxgx.gmail.com` | 110 |
  | AppIndicator and KStatusNotifierItem Support | `appindicatorsupport@rgcjonas.gmail.com` | 66 |
  | Just Perfection | `just-perfection-desktop@just-perfection` | 37 |
  | Blur my Shell | `blur-my-shell@aunetx` | 78 |
  | Caffeine | `caffeine@patapon.info` | 61 |
  | Clipboard Indicator | `clipboard-indicator@tudmotu.com` | 74 |
  | Vitals | `Vitals@CoreCoding.com` | 86 |
  | GSConnect | `gsconnect@andyholmes.github.io` | 74 |
  | Tiling Assistant | `tiling-assistant@leleat-on-github` | 56 |
  | Rounded Window Corners Reborn | `rounded-window-corners@fxgn` | 26 |

  10 个都是 EGO 下载 zip 手工解压装的（方式 B），uuid 已写进
  `org.gnome.shell enabled-extensions`，**但运行中的 shell 不会重扫扩展目录**，
  所以**要注销重登才真正生效**（Wayland 下不能原地重启 shell）。
  重登前 `gnome-extensions list` 一条都列不出来（系统自带的那几个也卸了），
  `gnome-extensions enable <uuid>` 会报「扩展不存在」。

  10 个扩展的 `metadata.json` 都声明支持 `51`（`rounded-window-corners@fxgn` 只支持 `51`）。

  **扩展管理器已装**：`com.mattjakeman.ExtensionManager` 0.7.0，Flatpak 用户级安装
  （`flatpak install --user`），运行时 `org.gnome.Platform/x86_64/51`。
  桌面项 `~/.local/share/flatpak/exports/share/applications/com.mattjakeman.ExtensionManager.desktop`
  存在，所以 **GNOME Shell 会检查扩展更新**了（shell 的 `updatesSupported` 要求
  `com.mattjakeman.ExtensionManager.desktop` 或 `org.gnome.Extensions.desktop` 存在，
  本机走的是前者；`gnome-shell-extension-app` 那个 GNOME 自带的「扩展」应用没装）。

  启动：`flatpak run com.mattjakeman.ExtensionManager`。

  2026-11-21 卸掉了 **User Themes**（`user-theme@gnome-shell-extensions.gcampax.github.com`）：
  试过 Lavanda / WhiteSur / Orchis 三套主题后确认 GNOME 45+ 换主题坑太多，
  详见 [gnome-themes.md](gnome-themes.md)。

  2026-11-21 卸掉了 **Fedora 自带的 5 个扩展**（系统包，方式 D）：
  apps-menu、background-logo、launch-new-instance、places-menu、window-list。
  连带删掉了 `gnome-classic-session` 和 `gnome-shell-extension-common`，
  **登录界面里的「GNOME Classic」选项因此消失**（硬依赖，拦不住，见上面「注意」）。

  `enabled-extensions` 现在就是上面那 10 个用户级 uuid，
  没有 `background-logo@fedorahosted.org` 这类已删包的残留，
  也没有 `gtk4-ding@smedius.gitlab.com`、`Rounded_Corners@lennart-k` 这种历史残留。

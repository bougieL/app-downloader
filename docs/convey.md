# Convey

- 仓库地址：https://gitlab.gnome.org/donnybeelo/convey
- GitHub 镜像（只读）：https://github.com/donnybeelo/Convey
- Flathub：https://flathub.org/apps/net.donnybeelo.Convey
- 作者 nightly 仓库：https://releases.donnybeelo.net/donnybeelo.flatpakrepo
- Debian 打包进度：https://bugs.debian.org/cgi-bin/bugreport.cgi?bug=1148981（ITP/RFP，尚未进官方仓库）

以「会话（conversation）」为中心的邮件客户端，GTK4 + libadwaita 原生界面，Vala + C 编写。
是 GNOME 官方 **Geary 的硬分叉（hard fork）**，与 GNOME 基金会无隶属关系。

## 为什么分叉

Geary 官方版多年未发稳定版（最后一个稳定版是 2024 年），Convey 接过代码继续开发，版本号自成一路
（Convey 51.0 时 Geary 才 46.0）。相对 Geary 增加的东西：

- 会话视图为核心：一线程一屏，邮件自动归拢
- 全文 + 关键词搜索
- Autocrypt / PGP 支持、发信签名
- 邮件列表（mailing list）管理、未读过滤、拼写检查、代理配置
- Microsoft 365 / Exchange 支持（经 DavMail）
- 桌面通知、新邮件提示

**支持的服务**：Gmail、Yahoo!、Outlook.com 等标准 IMAP/SMTP，以及 Microsoft 365。

## 技术栈里的 webview 部分（重要）

界面是原生 GTK4，但**渲染 HTML 邮件正文用的是 WebKitGTK WebView**（源码里
`src/client/conversation-viewer/conversation-web-view.vala`、`src/client/components/components-web-view.vala`，
基类明确写着 "Base class for all WebKit2 WebView instances"）。富文本编辑器也是 WebView。

所以准确说法是：**应用外壳与 UI 框架是原生 GTK，不是 Electron / Chromium 套壳**，
但显示 HTML 邮件时仍会拉起 WebKitGTK 进程。跟 Geary 同源，同理。

对比同样在 Flathub 的那些套壳客户端（metadata 里 base 是 `org.electronjs.Electron2.BaseApp`）：
Proton Mail、Tuta、BlueMail、Mailspring、Fastmail、ElectronMail；以及
Tauri/Wails/Capacitor 系的 Meron（Wails v3）、Aerion（Wails v2）、Aster Mail（Capacitor）。

## 安装

### Flatpak（推荐，本项目约定优先 flatpak）

```bash
flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
flatpak install flathub net.donnybeelo.Convey
```

系统级安装加 `sudo`；用户级安装加 `--user`。两者可以共存，但**同一 app id 别装两份**，
否则 `flatpak run` 会报 "already installed"。

**注意运行时体积**：Convey 应用本体只有 ~16 MB，但它依赖 `org.gnome.Platform/x86_64/51`，
这个 GNOME 运行时装完约 1.1 GB。本机已有 `org.freedesktop.Platform`（Steam/Blender 用的）和
`org.gnome.Platform/x86_64/47`，**GNOME 51 运行时是另开一份，不复用**。首次安装要下 1 GB 上下，
网速 800 KB/s 左右时约 20 分钟起，看着像卡住了其实在下载。

### nightly（作者自建 flatpak 仓库）

```bash
flatpak remote-add --if-not-exists donnybeelo https://releases.donnybeelo.net/donnybeelo.flatpakrepo
flatpak install donnybeelo net.donnybeelo.Convey
```

仓库 `DefaultBranch=nightly`，跟 Flathub 的 stable 并行。同一 app id 两边都有时，
flatpak 会按 remote 优先级选，nightly 版本号形如 `51.0~beta` / `51.0~devel`。

### 源码编译

依赖 Vala、GTK4、libadwaita、WebKitGTK、meson ≥ 0.63：

```bash
git clone https://gitlab.gnome.org/donnybeelo/convey.git
cd convey
meson setup _build --prefix=~/.local
meson compile -C _build
meson install -C _build
```

`--profile` 选项决定 app id 后缀（`release` / `beta` / `devel`），默认 `auto`：
在 git 仓库里检出时自动用 `devel`（app id 带 `.Devel`），否则报错。

## 配置与数据位置

flatpak 下都在 `~/.var/app/net.donnybeelo.Convey/`：

| 路径 | 内容 |
| --- | --- |
| `config/convey/` | 账号配置、GNOME Online Accounts 账号（`goa_account_*`）、拼写词典 |
| `data/convey/` | 邮件数据库、本地数据 |
| `data/convey/gnupg/` | OpenPGP 密钥环 |
| `data/folks/` | 联系人（folks）关系 |
| `data/keyrings/` | 密钥环 |
| `cache/convey/` | IMAP 数据库缓存、字体缓存 |

源码里 home 目录名由 `get_convey_home_dir_name()` 决定：release 构建或 flatpak 沙箱里是 `convey`，
devel 构建是 `convey-devel`——**不同构建的账号和邮件库不共享**。

非 flatpak 安装则是标准的 `~/.config/convey/`、`~/.local/share/convey/`、`~/.cache/convey/`。

## 命令行

flatpak 里用 `flatpak run net.donnybeelo.Convey <参数>`，或先 `flatpak run --command=convey ...`。
直接装的用 `convey`。

| 参数 | 说明 |
| --- | --- |
| `-d` | 打印调试日志 |
| `-i` | 开启 WebKitGTK Inspector（调试 HTML 邮件渲染用） |
| `-q` | 优雅退出 |
| `-n` | 开新窗口 |
| `-v` | 显示版本 |
| `--log-imap` | 记录 IMAP 网络活动 |
| `--log-smtp` | 记录 SMTP 网络活动 |
| `--log-sql` | 记录数据库查询（输出量很大） |
| `--log-conversations` | 记录会话监控 |
| `--log-folder-norm` | 记录文件夹同步（normalization） |
| `--log-replay-queue` | 记录 IMAP 变更重放队列 |
| `--log-deserializer` | 记录 IMAP 网络反序列化 |

位置参数收 `mailto:` URI，命令行发邮件：
`flatpak run net.donnybeelo.Convey 'mailto:someone@example.com'`。

## 更新

**沿用当初的安装方式**：

- Flathub 装的：`flatpak update net.donnybeelo.Convey`（系统级加 `sudo`）。
  整体更新是 `flatpak update`（不带参数会更新所有应用和运行时）。
- nightly 仓库装的：同样 `flatpak update`，但要先确认 remote 还在：
  `flatpak remotes --user`；仓库删了要重新 `flatpak remote-add`。
- 源码编译的：重新 `meson compile -C _build && meson install -C _build`。

**更新运行时不会丢数据**：邮件库和账号配置在 `~/.var/app/net.donnybeelo.Convey/` 里，
升级只换 `/var/lib/flatpak/app/` 下的程序本体。

## 检测当前安装方式

```bash
flatpak list | grep -i convey        # 有记录 → flatpak 装的
flatpak list --columns=application,installation | grep -i convey
command -v convey                     # 有 → 非 flatpak 装的（源码编译或发行版包）
```

判据：

- `flatpak list` 有 `net.donnybeelo.Convey` 且 installation 是 `system` / `user` → flatpak。
  再看来源 remote：`flatpak remote-info --user flathub net.donnybeelo.Convey` 的「来源」字段，
  是 `flathub` 就是正式版，是 `donnybeelo` 就是 nightly。
- `~/.local/bin/convey` 存在且 flatpak 里没有 → 源码编译装的。

## 注意

- **本机状态（2026-11-09 核实）**：Debian trixie / amd64，Wayland 会话，
  `net.donnybeelo.Convey` 51.0 装在 **system** 层（`sudo flatpak install -y flathub net.donnybeelo.Convey`）。
  首次安装拉了 `org.gnome.Platform/x86_64/51` 运行时，占盘约 1.1 GB。
- **flatpak 的 `--user` 和 system remote 会冲突**：`flatpak remote-info flathub <app>` 会报
  「在多个安装中发现远程仓库 flathub」并要求选择，得写 `flatpak remote-info --user flathub <app>`
  或 `flatpak remote-info --system flathub <app>` 明确指定。安装同理，命令里给 `--user` / `--system`。
- **flatpak 安装时 GNOME Online Accounts 会拉起授权窗口**，涉及 `org.gnome.OnlineAccounts`、
  `org.gnome.keyring.SystemPrompter`、`org.gnome.evolution.dataserver.*` 的 D-Bus 权限（见 metadata）。
  沙箱里 OAuth 回调走不通时要检查这几条 Session Bus 策略。
- 沙箱只共享 `xdg-desktop`，其他目录（`~/.ssh`、`~/.gnupg` 等）默认**不可见**。
  Convey 自己把 GPG 密钥环放在 `data/convey/gnupg/`，不需要额外挂载。
- 用 `-i` 开 WebKitGTK Inspector 调 HTML 邮件渲染；普通用户看不到这个开关。
- 首次启动会在 `~/.var/app/net.donnybeelo.Convey/` 下建 `config/`、`data/`、`cache/` 三棵树，
  里面有 `folks/`（联系人）和 `goa_account_*`（GNOME 在线账号），属正常现象。
- 邮件是按会话归拢的，收件方或主题不规则（各类自动通知、验证码）会散成很多「会话」，
  这是设计如此不是 bug。
- Convey 的 GitHub 仓库是**只读镜像**，改代码要往 GitLab 提：
  https://gitlab.gnome.org/donnybeelo/convey
- 作者 nightly 仓库的 GPG key 写在 `.flatpakrepo` 文件的 `GPGKey=` 里，flatpak 首次添加 remote 会自动导入。

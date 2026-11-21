# ChatAgent

- 仓库地址：https://github.com/bougieL/chat-agent-releases

Electron 桌面客户端，通过 GitHub Release 对外发布安装包。

## 资产与安装（按平台）

### Linux

| 架构 | rpm | deb | AppImage |
| --- | --- | --- | --- |
| x86_64 | `chat-agent-<版本>.x86_64.rpm` | `chat-agent_<版本>_amd64.deb` | `chat-agent-<版本>.AppImage` |

目前只发布 x86_64 资产，arm64 机器需要自己用源码构建或等待发布。

**优先用 rpm / deb**（包管理器装）：两者都装到 `/opt/ChatAgent/`，并注册系统桌面项
`/usr/share/applications/chat-agent.desktop` 和图标
`/usr/share/icons/hicolor/*/apps/chat-agent.png`。不想用 sudo 才退到 AppImage。

```bash
# Fedora / RHEL 系
sudo dnf install ./chat-agent-<版本>.x86_64.rpm
# Debian / Ubuntu 系
sudo apt install ./chat-agent_<版本>_amd64.deb
```

AppImage 不用 sudo：`chmod +x` 后直接运行，或放 `~/apps/chat-agent/` 再软链到 `~/.local/bin/chat-agent`。

#### 怎么判断当初是怎么装的

| 装法 | 识别特征 | 更新命令 |
| --- | --- | --- |
| rpm | `rpm -q chat-agent` 有输出 | `sudo dnf install ./chat-agent-<版本>.x86_64.rpm` |
| deb | `dpkg -l chat-agent` 有输出 | `sudo apt install ./chat-agent_<版本>_amd64.deb` |
| AppImage | `~/apps/chat-agent/chat-agent.AppImage` 存在 | 见下方「更新（AppImage 方式）」 |

rpm / deb 都往 `/opt/ChatAgent/` 装，靠上面两条包管理器命令区分。
AppImage 不会碰 `/opt/ChatAgent/`，它就是一个独立的可执行文件。

**更新前必须先确认装法，再沿用同一种方式更新**，别混着来。

#### 更新（rpm / deb 方式）

```bash
gh release download <版本> --repo bougieL/chat-agent-releases \
  -p "chat-agent-<版本>.x86_64.rpm" -D tmp/
sudo dnf install ./tmp/chat-agent-<版本>.x86_64.rpm
```

dnf 会自己识别成「升级同一个包」。升级前先退出正在运行的 ChatAgent，
否则装到一半会报文件被占用。

#### 更新（AppImage 方式）

AppImage 就地替换，不用装包：

```bash
gh release download <版本> --repo bougieL/chat-agent-releases \
  -p "chat-agent-<版本>.AppImage" -D ~/apps/chat-agent/
mv ~/apps/chat-agent/chat-agent-<版本>.AppImage ~/apps/chat-agent/chat-agent.AppImage
chmod +x ~/apps/chat-agent/chat-agent.AppImage
```

替换前先退出正在运行的 ChatAgent。AppImage 跑起来会有两类进程：
挂载点里的 `chat-agent`（真正的应用）和 runtime 本体（路径就是你执行的那个文件），
挂载目录在 `/tmp/.mount_chat-*`。替换完顺手把桌面项里的版本号同步一下：
`~/.local/share/applications/chat-agent.desktop` 的 `X-AppImage-Version=`。

**注意**：`gh release download` 下的是 `.AppImage` 时不会截断，但**别忘了 `chmod +x`**，
覆盖后权限会丢，表现是双击没反应、命令行报 `Permission denied`。

### macOS

资产：`chat-agent-<版本>.dmg`（挂载后拖 `ChatAgent.app` 进 `/Applications`）、`ChatAgent-<版本>-arm64-mac.zip`（免安装 `.app`，只有 arm64 版）。

macOS 端暂时只有 Apple Silicon（arm64）构建。

### Windows

资产：`chat-agent-<版本>-setup.exe`（安装器）。

## 忽略的文件

同一个 release 里还有 electron-updater 的元数据，别当安装包下载：

- `latest.yml` / `latest-linux.yml` / `latest-mac.yml`：自动更新索引
- `*.blockmap`：差分更新用
- `builder-debug.yml`：构建调试信息

## 注意

- 升级前先退出正在运行的 ChatAgent，否则安装器覆盖不了正在占用的文件。
- Electron 应用用 AppImage 运行时，文件本身要可执行（`chmod +x`），首次启动可能会被 AppArmor/flatpak 之类的沙箱拦一下。
- `latest-linux.yml` 里的 sha512 只对 AppImage 有效，对 `.deb` 无效（记录的大小和哈希都对不上实际发布
  的 deb，估计是打包后重传过的）。校验 deb 别拿 yml 比，直接用 `dpkg-deb -I <包>` 看 Version 对不对。
- `chat-agent --version` 不会打印版本号，会直接拉起 GUI 主进程并挂住。
  看版本：rpm 装用 `rpm -q chat-agent`；deb 装用 `dpkg -l chat-agent`；
  AppImage 装看桌面项 `~/.local/share/applications/chat-agent.desktop` 的 `X-AppImage-Version=`。
- **rpm 包没有签名**，用 `rpm -K` 校验会报「摘要 SIGNATURES 不正确」，这是正常的
  （`rpm -qip` 里 `Signature: (none)`）。别当成下载坏了，也别因此去加 `--nogpgcheck` 之类的绕过，
  dnf 装本地 rpm 时只会提示「skipped OpenPGP checks for 1 package from repository: @commandline」，可以忽略。
  校验完整性用 `rpm -qip <包>` 看 Version 对不对。
- **启动时终端会打一条 WARN**，不用管：
  `[claude-replay] unmapped transcript row type "tools-used" — the CLI's storage format may have drifted; the row was ignored`
  这是 Claude Code CLI 那边 transcript 存储格式变了，ChatAgent 跳过读不懂的那一行，
  不影响启动。真正的错误看 `~/.config/ChatAgent/logs/<日期>-error.log`。
- **`gh release download` 偶尔会下出截断的包**：`dpkg-deb -I <包>` 还能正常读出 Version，
  但 `apt install` 报「错误： 归档文件中成员文件头无效 / 无法从 ... 读取元数据」。
  校验要连数据段一起验，`dpkg-deb -I` 只读控制段不够：

  ```bash
  dpkg-deb -c <包>.deb > /dev/null && echo OK
  ```

  不 OK 就删掉重新下载。
- **`gh release download` 也会直接报网络错误**（2026-11-10 实测）：
  `Get "https://api.github.com/repos/<owner>/<repo>/releases/tags/<tag>": EOF`。
  这是 GitHub API 侧的临时故障，**重跑同一条命令通常就成功了**，不用换下载方式
  （curl 直连 release 资产也可以，但 `gh release download` 更稳，它自带重试）。
- **本机状态（2026-11-21 核实）**：Fedora 45 / amd64，**走 rpm 路线**（不是 deb、不是 AppImage）：
  - `rpm -q chat-agent` → `chat-agent-1.46.0-1.x86_64`（2026-11-15 发布，当前最新）
  - 程序装在 `/opt/ChatAgent/`，主程序 `/opt/ChatAgent/chat-agent`
  - 桌面项：`/usr/share/applications/chat-agent.desktop`（`Exec=/opt/ChatAgent/chat-agent %U`）
  - 图标：`/usr/share/icons/hicolor/*/apps/chat-agent.png`
  - 命令行入口：`~/.local/bin/chat-agent` → `/opt/ChatAgent/chat-agent` 的软链
    （rpm **不提供** `/usr/bin/chat-agent`，自己软链了一个）
  - 早先的 AppImage 装法已卸掉：`~/apps/chat-agent/`、`~/.local/share/applications/chat-agent.desktop`、
    `~/.local/share/icons/` 下的本地图标都已删除
  - rpm 版**不需要 FUSE**（不像 AppImage），启动也没有 FUSE 那个 ~100% CPU 的开销
  - 但 `fuse-libs` 仍留着：同机的 obsidian / tessera 也是 AppImage，要用 `libfuse.so.2`

### AppImage 起不来：缺 libfuse.so.2

> 只影响 **AppImage** 装法。rpm / deb 装的是普通目录，不走 FUSE，不会有这个问题。

**现象**（2026-11-21 实测，Fedora 45）：双击桌面图标没反应，命令行启动立刻退出并打印：

```
dlopen(): error loading libfuse.so.2

AppImages require FUSE to run.
You might still be able to extract the contents of this AppImage
if you run it with the --appimage-extract option.
```

**原因**：AppImage 运行时靠 FUSE 挂载自带的 squashfs，需要 FUSE 2 的共享库 `libfuse.so.2`。
Fedora 的 `fuse` 包（2.9.9）**只提供** `/usr/bin/fusermount`、`/usr/bin/mount.fuse`、
`/usr/bin/ulockmgr_server`，**不含 `libfuse.so.2`**——库在单独的 `fuse-libs` 包里。
所以只装 `fuse` 没用，`fuse3` / `fuse3-libs` 提供的 `libfuse3.so.3` AppImage 也不认。

**解决**：

```bash
sudo dnf install fuse-libs
```

装完 `/usr/lib64/libfuse.so.2` 就有了，**不用重装 AppImage**，直接再启动一次即可。
检查：`rpm -q fuse-libs` 有输出、`ls /usr/lib64/libfuse.so.2` 存在。

没有 sudo 权限时可以绕开 FUSE：

```bash
# 方式一：解包后直接跑
~/apps/chat-agent/chat-agent.AppImage --appimage-extract
./squashfs-root/AppRun

# 方式二：运行时自动解包（每次启动都解到临时目录，慢一点）
APPIMAGE_EXTRACT_AND_RUN=1 ~/apps/chat-agent/chat-agent.AppImage
```

**启动瞬间 AppImage runtime 会跑满一个核**，不用当成故障：
`ps` 里 `~/.local/bin/chat-agent`（即 runtime 本体）显示 ~100% CPU、
`wchan` 是 `fuse_dev_do_read`，那是 Electron 通过 FUSE 读 asar / resources 的正常开销，
稳定后实测会降到 0%。空闲的 `--appimage-mount` 挂载更是全程 0%。

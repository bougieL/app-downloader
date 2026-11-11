# 网易云音乐 GPUI (netease-music-gpui)

- 仓库地址：https://github.com/bougieL/netease-music-gpui
- 作者：bougieL

用 Rust + [GPUI](https://gpui.rs/) 复刻的网易云音乐桌面客户端，支持扫码登录、推荐页、个人歌单、播放队列、心动模式、歌词与评论等。目前处于开发中，**只有 nightly 版本**（tag 形如 `nightly-<版本>-<日期>`，GitHub 上标记为 pre-release），主要在 macOS 上开发验证，其他平台未验证。

## 资产与安装（按平台）

### Linux

资产：`netease-music-gpui-linux-x64.tar.gz`（目前只有 x64）。

解压出 `netease-music-gpui/` 目录，里面是可执行文件 `netease-music-gpui` 和 `assets/`（`icons/`、`images/`、`font/`）。**必须整个目录一起放**，只复制可执行文件会缺资源。

```bash
tar -xzf netease-music-gpui-linux-x64.tar.gz
mv netease-music-gpui ~/apps/
ln -sf ~/apps/netease-music-gpui/netease-music-gpui ~/.local/bin/netease-music-gpui
```

Linux 运行需要系统提供 ALSA 和 Vulkan 运行库。

桌面入口写到 `~/.local/share/applications/netease-music-gpui.desktop`，`Exec` / `TryExec` 用可执行文件的绝对路径，`Icon` 用图标名（`netease-music-gpui`）。写完跑 `update-desktop-database ~/.local/share/applications/`。

**图标**：Linux 产物里没有 PNG/ICO 图标，`assets/icons/logo/logo.svg` 的圆形是 `fill="none"`（在应用内由 GPUI 填色），直接拿来当桌面图标会是一个没有红底的空图标。要用真正的红底图标，从仓库的 `packaging/163Music.icns` 里取（icns 内嵌 PNG，按 `ic07`=128、`ic08`=256、`ic09`=512、`ic10`=1024 分块存放）：

```bash
curl -sL -o 163Music.icns https://raw.githubusercontent.com/bougieL/netease-music-gpui/main/packaging/163Music.icns
python3 - <<'PY'
import struct
d = open('163Music.icns', 'rb').read()
off = 8
while off < len(d):
    t, l = struct.unpack('>4sI', d[off:off+8])
    payload = d[off+8:off+l]
    if payload[:8] == b'\x89PNG\r\n\x1a\n':
        open(f'icns_{t.decode()}.png', 'wb').write(payload)
    off += l
PY
for pair in 128:ic07 256:ic08 512:ic09; do
  s=${pair%%:*}; chunk=${pair##*:}
  d=~/.local/share/icons/hicolor/${s}x${s}/apps
  mkdir -p "$d"
  cp "icns_${chunk}.png" "$d/netease-music-gpui.png"
done
gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor
```

macOS 的 `.app` 自带 `163Music.icns`，不用管。

数据目录：`$XDG_DATA_HOME/netease-music-gpui/`（默认 `~/.local/share/netease-music-gpui/`），`session.cookie` 保存登录 Cookie（未加密，权限 `0600`，不要分享）。

### macOS

资产：`netease-music-gpui-macos-arm64.zip`、`netease-music-gpui-macos-x64.zip`。解压出 `.app`，拖进 `/Applications`。命令行入口是 `.app/Contents/MacOS/` 下的可执行文件，需要时可软链到 `~/.local/bin/netease-music-gpui`。

数据目录：`~/Library/Application Support/NeteaseMusicGpui/`。

### Windows

资产：`netease-music-gpui-<版本>-x86_64-pc-windows-msvc.zip`（免安装），以及同名 `.zip.sha256` 校验文件。

ZIP 内是一个完整目录：

```text
netease-music-gpui/
  netease-music-gpui.exe
  assets/
  *.dll          # VC release 运行库
  README.txt
  build-info.json
```

完整解压后直接双击 EXE 即可，无需另装 VC 运行库。数据目录 `%APPDATA%/NeteaseMusicGpui/`，缓存在 `%LOCALAPPDATA%/NeteaseMusicGpui/Cache/`。升级时从托盘退出旧程序，再解压新版到新目录。

## 注意

- 没有正式版 release，只有 nightly 预发布；查版本用 `gh api repos/bougieL/netease-music-gpui/releases --jq '.[].tag_name'`，不要用 `releases/latest`（可能拿不到或拿到预发布）。`releases/latest` 在只有预发布时通常会返回预发布，但优先按 tag 列表取最新的那个。
- Linux 启动时日志会报 `Failed to load bundled font fonts/ibm-plex-sans/...` 和 `fonts/lilex/...`：Linux 产物只带了 `assets/font/dolphin*.ttf`，没打包 GPUI SVG 渲染器要的那两个字体，属于已知噪音，不影响使用（SVG 里的文字会回退到系统字体）。
- 各平台资产名里的版本号一致（如 `0.1.0`），但 tag 每天一个，不要用资产名里的版本当版本标识。

- Linux 产物里**没有** `build-info.json`（Windows 包才有），所以装的是哪个 nightly 光看文件看不出来。
  判断是否已是最新，只能拿新旧二进制比哈希：

  ```bash
  sha256sum ~/apps/netease-music-gpui/netease-music-gpui
  # 与新下载的 nightly 解包出来的二进制比对，一致就说明已经是最新版
  ```

- 图标那段里的 `163Music.icns` 从 `raw.githubusercontent.com` 下载，这个域名不一定可达（实测会超时）；
  拿不到时改走 API 取同一份文件：

  ```bash
  gh api repos/bougieL/netease-music-gpui/contents/packaging/163Music.icns --jq '.content' | base64 -d > 163Music.icns
  ```

- 可执行文件没有 `--version` 这类参数，命令行带任何参数运行都会直接拉起 GUI（把 `--version` 当版本查询
  会在终端挂住并弹出窗口）；判断装的是哪个 nightly 只能用上面的哈希比对。
- Linux 图标实测不来自 `assets/icons/logo/logo.svg`，icns 里的分块是 `ic07`=128、`ic08`=256、`ic09`=512、
  `ic10`=1024，另有 `ic11`–`ic14` 更大尺寸，按需取用。
- **本机状态（2026-11-10 核实）**：装的是 `nightly-0.1.0-20261103`（上游当时最新的 nightly），
  二进制 sha256 `642e2c73ee1b80215762c1400e726af54e47395d305cbb910cfab5c974350093`。
  下载最新 nightly 解包后二进制哈希一致、`assets/` 目录 `diff -rq` 也无差异，即已是最新，**无需更新**。
  比对 assets 目录用：

  ```bash
  diff -rq ~/apps/netease-music-gpui/assets <解包目录>/netease-music-gpui/assets
  ```

- **启动器里图标显示为空白（缺图标）**：多半是 `netease-music-gpui.desktop` 的 `Icon` 写成了指向不存在
  文件的绝对路径。例如写成 `Icon=~/apps/netease-music-gpui/assets/icons/netease-music-gpui.png`——
  产物里根本没有这个 PNG；或写成 `Icon=~/apps/.../assets/icons/logo.svg`——真实的 svg 在
  `assets/icons/logo/logo.svg`（多一层 `logo/` 子目录），少写一层同样指向空文件，GNOME 都解析不到图标。
  现象是启动器/dock 上一个默认空白方块，应用本身能正常打开。

  排查用 `cat ~/.local/share/applications/netease-music-gpui.desktop` 看 `Icon` 行，再 `ls` 那个路径确认
  文件是否真的存在；按图标主题规范装的 PNG 在 `~/.local/share/icons/hicolor/<尺寸>x<尺寸>/apps/` 下。

  解决：**不要写绝对路径**，按上面「图标」一节把 PNG 装进 hicolor 主题后，把 `Icon` 改成纯图标名
  `netease-music-gpui`（不带路径、不带扩展名），这是 GNOME/freedesktop 最可靠的解析方式。改完刷新缓存：

  ```bash
  gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor
  update-desktop-database ~/.local/share/applications/
  ```

  若仍是旧空白，是 dock/启动器缓存了旧图标，注销重登（X11 下 `Alt+F2` 输入 `r`）或取消收藏再重新收藏即可。
  确认图标名是否已进缓存：`grep -a -c netease-music-gpui ~/.local/share/icons/hicolor/icon-theme.cache`。

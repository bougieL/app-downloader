# PhotoCraft

- 仓库地址：https://github.com/storytold/photocraft
- 官网：https://getartcraft.com/apps/photocraft

用纯 Rust 从零写的开源图像编辑器（Photoshop 的 clean-room 重制），支持图层、蒙版、调整图层、
文字、矢量、笔刷以及 PSD / PSB 读写。走 GitHub Release，每个 release 同时提供桌面客户端、
命令行工具（`photocraft-cli`）与 WebAssembly 版资产。项目仍处于早期 alpha。

每个资产名里的 `<版本>` 是版本号（如 `0.20.2`），tag 带 `v` 前缀（如 `v0.20.2`）。
release 附 `SHA256SUMS.txt` 与 `SHA256SUMS.txt.asc`、各资产同名 `.asc` 签名，
校验用 `sha256sum -c SHA256SUMS.txt`。

## 资产与安装（按平台）

### Linux

按 `uname -m` 选架构，格式任选其一：

| 格式 | x86_64 | aarch64 | 说明 |
| --- | --- | --- | --- |
| AppImage | `photocraft-<版本>-linux-x86_64.AppImage` | `photocraft-<版本>-linux-aarch64.AppImage` | 免安装，可用同名 `.zsync` 增量更新 |
| Flatpak | `photocraft-<版本>-linux-x86_64.flatpak` | `photocraft-<版本>-linux-aarch64.flatpak` | 沙箱安装 |
| deb | `photocraft-<版本>-linux-x86_64.deb` | `photocraft-<版本>-linux-aarch64.deb` | Debian/Ubuntu |
| rpm | `photocraft-<版本>-linux-x86_64.rpm` | `photocraft-<版本>-linux-aarch64.rpm` | Fedora/RHEL/openSUSE |
| tar.gz | `photocraft-<版本>-linux-x86_64.tar.gz` | `photocraft-<版本>-linux-aarch64.tar.gz` | 解压即用 |

另有 riscv64 的 `photocraft-<版本>-linux-riscv64.tar.gz`（仅 tar.gz，基于 Ubuntu 24.04 构建，需 glibc 2.39+），
以及给 Arch 的 `photocraft-<版本>-linux-<架构>.pkg.tar.zst`。

**优先用 Flatpak**（本项目约定安装软件优先 flatpak）：

```bash
flatpak install --user photocraft-<版本>-linux-x86_64.flatpak
flatpak run ai.storyteller.photocraft
```

flatpak bundle 需要来自 Flathub 的 freedesktop runtime，`flatpak` 会在安装时一并提示安装。

系统级安装（deb / rpm，需要 sudo）：

```bash
sudo apt install ./photocraft-<版本>-linux-x86_64.deb
# 或对应发行版的 rpm 包管理器
```

不想用 sudo 就用 AppImage：`chmod +x` 后直接运行，或放 `~/apps/` 再软链到 `~/.local/bin/photocraft`。
AppImage 首次运行会把启动器图标和菜单项注册进 `~/.local/share`，设 `PHOTOCRAFT_NO_DESKTOP_INTEGRATION=1` 可跳过。

### macOS

- 桌面应用：`photocraft-<版本>-macos-universal.dmg`（universal，Apple silicon + Intel，已签名公证）。
  挂载后把 `PhotoCraft.app` 拖进 `/Applications`。
- 命令行工具：`photocraft-cli-<版本>-macos-universal.zip`，用 `ditto -x -k` 解压，
  把 `photocraft-cli` 放进 `~/.local/bin`。裸二进制不带 stapled 公证票据，首次运行会联网校验公证。

### Windows

| 架构 | 安装器 | 免安装 |
| --- | --- | --- |
| x64 | `photocraft-<版本>-windows-x64.msi` | `photocraft-<版本>-windows-x64-portable.zip` |
| arm64 | `photocraft-<版本>-windows-arm64.msi` | `photocraft-<版本>-windows-arm64-portable.zip` |
| x86 | `photocraft-<版本>-windows-x86.msi` | `photocraft-<版本>-windows-x86-portable.zip` |

安装器和可执行文件均已代码签名。另有 `photocraft-<版本>-windows-<架构>-symbols.zip`（调试符号，别装）。

### FreeBSD

`photocraft-<版本>-freebsd-x86_64.tar.gz`，布局按 `/usr/local` 组织，先装运行库再解包：

```bash
pkg install libxkbcommon wayland libX11 libXcursor libXrandr libXi libxcb mesa-libs vulkan-loader gtk3 fontconfig freetype2 alsa-lib
tar -xzf photocraft-<版本>-freebsd-x86_64.tar.gz --strip-components 1 -C /usr/local
```

### Web

`photocraft-web-<版本>.zip` 是 WebAssembly 版，自托管见仓库 `packaging/web/README.md`，不走本地安装。

## CLI 与 MCP

`photocraft-cli` **不通过 npm / pnpm / brew / mise 等包管理器分发**，它捆在桌面程序里，不要去包管理器里找。

- Linux：AppImage 直接带 CLI，用 `./photocraft-<版本>-linux-x86_64.AppImage --cli --help` 调用；
  把可执行文件软链成名为 `photocraft-cli` 的链接也会自动切到 CLI 模式。`--cli mcp` 在 stdio 上起 MCP server。
- macOS：单独下载 `photocraft-cli-<版本>-macos-universal.zip`。
- Windows：CLI 随桌面安装一起提供。

UI、CLI、JSON 控制通道和 MCP server 共用同一套命令注册表，例如：

```bash
photocraft-cli run wave.psd --cmd filter.sharpen.smartSharpen --params '{"amount":80}' --out wave-final.png
photocraft-cli batch --actions grade.json --in ./raw --out ./graded
```

## 检测当前安装方式与更新

CLI 随桌面程序走同一份资产，所以更新就是更新桌面程序，沿用当初装的格式，不要换：

- Flatpak：`flatpak update ai.storyteller.photocraft`（或重新 `flatpak install --user` 新版 bundle）。
- deb / rpm：用系统包管理器装新包（升级会替换旧版）。
- AppImage：用同目录的 `.zsync` 走 AppImageUpdate 增量更新，或下载新 AppImage 替换。
- tarball：重新解压覆盖 `~/apps/<目录>` 并重建软链。
- macOS：下载新的 dmg 覆盖 `/Applications` 里的 `.app`；CLI 另行下载新 zip 替换。
- Windows：运行新 msi，或解压新 portable zip 到新目录。

判断不出当初用哪种格式装的（例如 `which photocraft` 指向 `~/.local/bin` 软链），
顺着软链 `readlink -f` 看目标是 AppImage、解压目录还是系统包，再按对应方式更新；
`flatpak list | grep -i photocraft` 能确认是不是 Flatpak 装的。实在判断不出来就问用户，不要猜、不要换方式重装。

## 注意

- 查版本用 `gh api repos/storytold/photocraft/releases/latest --jq '.tag_name'`，不要用资产名里的版本号当 tag。
- 签名/校验类文件不要选成安装包：`.asc`、`.sig`、`SHA256SUMS.txt`、`*-symbols.zip`、
  `*.oci-manifest.json`、`*.oci.sig` 都是校验或元数据文件。
- **同名项目很多**（另有 AI 换脸网站、iOS 摄影 App、教学小项目等）。认准 `storytold/photocraft`
  这个纯 Rust 桌面编辑器仓库，别下错到同名仓库。
- Wayland 下拖放文件到窗口暂时打不开（winit 0.30 的限制），用 File › Open，或复制图片后 Ctrl+V 粘贴；
  需要拖放可用 `WAYLAND_DISPLAY= ./photocraft-<版本>-linux-x86_64.AppImage` 走 XWayland。
- 项目仍是早期 alpha，缺少生成式 AI 功能、部分工具和插件兼容，别当 Photoshop 完全替代。
- Gentoo 有社区维护的 `media-gfx/photocraft-bin`（::snakebyte overlay），非官方团队维护，
  走 overlay 装的就沿用 `emerge` 更新，不要混用上面的 GitHub 资产。

- **flatpak 装的这版来自 `photocraft-origin` 远程仓库，该仓库可能已被禁用**：
  `flatpak remote-ls --updates` 什么都查不到，`flatpak update` 因此拉不到新版
  （报 `Can't fetch summary from disabled remote 'photocraft-origin'`）。
  更新只能下载 release 里的 `.flatpak` bundle 手动装到**当初那个 scope**
  （系统级装的就用 `--system`，用户级用 `--user`，别混着来，否则同一个 app id 装两份）：

  ```bash
  flatpak install --system photocraft-<版本>-linux-x86_64.flatpak
  ```

  装之前用 `SHA256SUMS.txt` 校验：`sha256sum -c SHA256SUMS.txt`。
  没下 `.asc` 签名文件时会报「没有那个文件或目录 / 警告：1 个列出的文件无法读取」，
  忽略那一条即可，flatpak 本体校验通过就行。

# Longshot（Switch 2 时之笛 PC 静态重编译版）

- 仓库地址：https://git.magicbean.dev/MagicBeanDev/Longshot（**Forgejo 自建 git**，不是 GitHub）
- 文档站：https://longshot.magicbean.dev
- 许可证：GPL-3.0-or-later
- 上游前身是 [Hookshot](https://github.com/NotRexed/Hookshot)（2026-11 被 DMCA 下架），
  维护者转移到了这个项目。

把《塞尔达传说：时之笛》的 Switch 2 版从静态重编译成原生 Windows / Linux x86-64 程序，
配 Vulkan 渲染器。**release 里不含游戏数据、密钥或可执行文件**：
安装器在玩家自己的机器上从自己的游戏副本编译出来。

## 两个东西，别搞混

| 东西 | 仓库 | 是什么 |
| --- | --- | --- |
| **Longshot Launcher** | `MagicBeanDev/LongshotLauncher` | 图形启动器，负责下载 builder、构建、更新游戏 |
| **Longshot Builder** | `MagicBeanDev/Longshot`（release 资产） | 无窗口，**只有命令行模式**；Launcher 就是调它来构建的 |

Launcher 的 AppImage 单独下；Builder 是 `MagicBeanDev/Longshot` 的 release 资产
（`Longshot-<版本>-x86_64.AppImage`）。直接双击 Builder 会弹一个 zenity 提示
「The Longshot Builder has no window of its own」，它只接受 `--cli`。

## Release 资产

| 仓库 | 资产 |
| --- | --- |
| `MagicBeanDev/Longshot` | `Longshot-<版本>-x86_64.AppImage`（Builder）、`SHA256SUMS`、`SHA256SUMS.sig`、`tools.lock` |
| `MagicBeanDev/LongshotLauncher` | `LongshotLauncher-<版本>-x86_64.AppImage`（+ Windows 版）|

Builder 的 release 里有 `SHA256SUMS`，**下载后必须校验**：

```bash
curl -sLO https://git-cdn.magicbean.dev/MagicBeanDev/Longshot/releases/download/<版本>/Longshot-<版本>-x86_64.AppImage
curl -sLO https://git-cdn.magicbean.dev/MagicBeanDev/Longshot/releases/download/<版本>/SHA256SUMS
grep 'x86_64.AppImage' SHA256SUMS | sha256sum -c -
```

查最新版本（Forgejo 的 API 和 GitHub 一样）：

```bash
gh api repos/MagicBeanDev/Longshot/releases --jq '.[0] | "\(.tag_name)\t\(.published_at)"'
gh api repos/MagicBeanDev/LongshotLauncher/releases --jq '.[0].assets[].name'
```

**渠道**：正式版（`v3.0.0`）、候选版（`v3.0.0-rc.x`）、每晚构建（`nightly-xxx-xxxxxxx`）。
最新正式版和最新 nightly 差好几版，**玩就选正式版**；
nightly 未测试，可能弄坏存档和设置。

## 安装（Linux）

```bash
mkdir -p ~/.local/opt/longshot
# Launcher（图形界面，负责构建和更新）
gh release download <版本> --repo MagicBeanDev/LongshotLauncher \
  --pattern 'LongshotLauncher-<版本>-x86_64.AppImage' --dir ~/.local/opt/longshot --clobber
chmod +x ~/.local/opt/longshot/LongshotLauncher-<版本>-x86_64.AppImage
```

图形流程：启动 Launcher → 按 **Build** → 选渠道（Stable）→ 选游戏文件
（`.nup`，或直接指到含 `romfs/` + `exefs/` 的目录）→ 选安装目录 → Next → 等构建。

## 命令行（Builder）

```bash
# 构建
./Longshot-<版本>-x86_64.AppImage --cli --nup ~/your-game.nup --out ~/Games/Longshot
# 从已有游戏目录更新（重新构建，保留存档和设置）
./Longshot-<版本>-x86_64.AppImage --cli --update ~/Games/Longshot
# 玩
./Longshot-<版本>-x86_64.AppImage --play ~/Games/Longshot
# 只检查游戏版本
./Longshot-<版本>-x86_64.AppImage --cli --check
```

常用参数：`--romfs` / `--exefs`（分开给目录）、`--game`（给一个同时含
`romfs/` 和 `exefs/` 的目录）、`--jobs <n>`（或 `LONGSHOT_JOBS=<n>`）、
`--log <file>`、`--compile-shaders {most,all}`、`--no-prepare-shaders`、
`--language <代码>`。

## 注意

- **要自己带游戏**：程序 ID `0400486024DEC000`，更新 `v131072`，build `e0f3ccfa`。
  要求 Vulkan 1.3、4GB 显存（推荐 12GB + mesh shader）、16GB 内存、构建期约 60GB 空间。
- **构建工具链会自动下载**：`emsdk` / `ffmpeg` / `sdl2` / `sysroot` / `cpm`，
  下到 `~/.local/share/longshot/builder`，第一次跑要等一会儿。
- **更新比首次快**：Launcher 保留游戏文件，更新时不用再给游戏。
  更新会重做 shader 准备，第一次启动会慢。
- **保留的东西**：存档（`~/.local/share/longshot`）、`longshot_options.ini`（设置和按键）、
  `longshot_env.txt`、`mods/`、`mods.json`；
  **会替换**：`bin/`、`image.bin`、shader 列表、`language.txt`、`texts/`。
- **游戏目录里的 `longshot` 脚本**是给双击用的：它先找 `longshot_version.txt` 里的
  `builder=` 行去跑 builder（`--launch <目录>`，直接开玩），builder 没了才退回到
  跑 `bin/longshot` 本身。**所以游戏目录必须和 builder AppImage 同盘可访问**。
- **Linux 上 NVIDIA DLSS 只能从纯 ASCII 路径的目录加载**（游戏目录名带中文就不行，
  得把 `libnvidia-ngx-dlss*.so.*` 复制到 `~/.local/share/longshot/ngx`）。
- **AMD FSR 4 在 Linux 没有官方库**，得自己放 `amd_fidelityfx_upscaler_vk.so`；
  FSR 3.1 每个版本都自带。
- **要 Vulkan 1.3**，Mesh shader 缺失会退回兼容路径。

## 本机状态（2026-11-20 核实）

- Fedora 45 / amd64，12 核，AMD Radeon RX 9070。
- **游戏版本 v3.0.0**（`source=bd6a0cdd`），装在独立挂载点的 `Games/Longshot` 下
  （挂载点路径按本机情况替换）。
  Builder `~/.local/opt/longshot/Longshot-v3.0.0-x86_64.AppImage`，
  Launcher `~/.local/opt/longshot/LongshotLauncher-v1.0.0-x86_64.AppImage`（SHA256 都校验过）。
- 从旧的 v3.0.0-rc.3 用 `--cli --update` 原地升级，**build 全部重做**：
  lift（1764 万条指令 100% 翻译，294522 个函数）→ 编译 → shader 准备
  （32390 条 pipeline，12 线程约 385 秒），整轮大约 25 分钟。
  `--work` 指向旧的构建缓存目录能复用 objcache，编译快很多。
- 桌面入口：`~/.local/share/applications/longshot.desktop`（图标 `longshot`），
  另有 `longshot-launcher.desktop`（更新/重建用）。

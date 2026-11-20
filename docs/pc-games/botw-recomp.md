# Savage Wind Recomp（塞尔达传说：旷野之息 PC 静态重编译版）

- 仓库地址：https://github.com/Revivedx/SavageWindRecomp
- 许可证：MIT
- 相关但**不同项目**的注意：[totk-pc-recomp](totk-recomp.md) 是**王国之泪**的重编译版，
  和本项目（旷野之息）不是一回事。

把《塞尔达传说：旷野之息》从 Switch 静态重编译成原生 Windows / Linux x86-64 程序，
配 Vulkan 渲染器。**release 里不含游戏数据，也不含任何从游戏派生的代码**：
安装器在玩家自己的机器上把游戏编译出来。

## 游戏版本

**只支持 1.9.0**（2026-02 之后的版本，含 Switch 2 上的泰语文本）。
1.8.2 在 Alpha 3 时还支持，**0.5.x 已经彻底放弃**：
上游的代码生成器和引擎都按补丁方式改游戏代码，只能保住一个版本。
用 `main` 模块 NSO 头的前 8 字节（build id）区分版本，1.9.0 是 `cd57b23fa4bbad65`。

上游 issue #633 里的报错就是这个意思：「你贴的是 1.8.2 的游戏文件」。

## Release 资产

| 资产 | 说明 |
| --- | --- |
| `Savage_Wind-<版本>-x86_64.AppImage` | Linux amd64，免安装 |
| `Savage_Wind-<版本>-windows.zip` | Windows，就地解压 |

AppImage 的启动画面是它自己的美术；玩家自己的图片放数据目录下的 `splash.jpg` / `splash.png` 会替换它。

## 安装（Linux）

AppImage 免安装（不需要 sudo）：

```bash
mkdir -p ~/Applications
gh release download <tag> --repo Revivedx/SavageWindRecomp \
  --pattern 'Savage_Wind-<版本>-x86_64.AppImage' --dir tmp/savagewind --clobber
chmod +x tmp/savagewind/Savage_Wind-<版本>-x86_64.AppImage
```

跑起来是图形界面：

- 第一次启动会问**游戏文件**：选自己解出来的目录（里面要有 `exefs/` 和 `romfs/`），
  它会校验版本、复制文件、把 RomFS 打包成镜像，**在本机重新编译游戏代码**（12 核约 40 分钟）。
- 之后每次启动直接进游戏。
- `--config`（或 AppImage 的「Savage Wind Configuration」菜单项）打开配置界面：
  Play、Settings、Game Files、Quit。
- 存档、设置、shader 缓存都在 `~/.local/share/savagewind`。

## 从源码构建（Linux）

Debian 13 / Ubuntu 25.04+ 的包：

```bash
sudo apt install clang lld make pkg-config python3-venv libsdl3-dev libavformat-dev libavcodec-dev \
    libavutil-dev libswscale-dev libvulkan-dev libshaderc-dev zlib1g-dev glslc vulkan-tools
```

（clang 要 18 或更新：提升出来的 FP/SIMD 代码用了 `__builtin_elementwise_*`；
SDL3 要 3.2 或更新。）

然后：

```bash
python3 -m venv tools/venv && tools/venv/bin/pip install lz4 capstone unicorn
make -j         # 只出启动画面（还没有游戏代码）
make gen        # 从自己的游戏文件重新编译游戏代码
make -j         # -> build/savagewind 和游戏代码库 build/swgame.so
```

跑：`./build/savagewind`（仓库根目录）。**当前目录有 `extracted/` 时就用它**
（开发模式），否则用 `~/.local/share/savagewind`。

`extracted/` 放自己解出来的游戏文件：

```
extracted/
  exefs/   main, main.npdm, rtld, sdk, subsdk0
  romfs/   完整的 RomFS（本体 + 更新合并）
```

`make gen` 会按 `main` 模块的 build id 选对应版本的补丁，
所以**换游戏版本必须重新 `make gen`**，旧 C 代码会被清掉。

## 游戏文件怎么解

上游不提供游戏文件，也不解密。玩家从自己的主机上解出来即可。
只有 NSP / XCI 时，用 hactool 合并本体和更新（更新包是 BKTR 补丁）：

```bash
# 1. 本体和更新各自导出成独立 .nca（程序 NCA：本体带 RomFS，更新带 BKTR 段）
# 2. 合并（section 序号要用更新的 BKTR 那一段，写 0 会存成更新的 ExeFS）
hactool -x -y --basenca=base.nca --section1=merged/romfs.bin update.nca
```

`merged/romfs.bin` 是合并后的 RomFS 镜像；`extracted/romfs/` 的文件树可以从镜像里按索引导出。

## 注意

- **RomFS 的文件数要完整**：完整的 1.9.0（本体 + 更新合并）有 **41062 个文件**，
  总字节 15129626830。少一个文件（少 180 个也会）游戏会在第一帧前崩。
  上游的设置界面会校验这个数，issue #686 就是少了一个文件导致 setup 拒绝。
- **hactool 认不出改过名的容器**：对重打包的 NSP / XCI 会报
  `Invalid NCA header! Are keys correct?`。用 `-t pfs0` / `-t xci` 指定类型，
  或者把里面的 NCA 拆成独立文件再喂给它。
- **keys 文件里的 17 字节值**：Lockpick 写的 `master_kek_source_*` /
  `mariko_master_kek_source_*` 是 16 字节密钥后面跟一个 NUL，hactool 只接受 16 字节，
  报 `Key (...) must be 32 hex digits!` 或直接退出。把尾部 NUL 去掉再给 hactool。
- **1.9.0 更新包是 titlekey crypto，title.keys 里可能缺它的 rights id**：
  本体 XCI 是 standard crypto（prod.keys 就能解），但 1.9.0 更新 NSP 的 NCA 是
  `Encryption Type: Titlekey crypto`，`Rights ID: 01007EF00011E8000000000000000003`。
  title.keys 里没有这条时，hactool 报
  `[WARN]: Unable to match rights id to titlekey. Update title.keys?`，
  解出来的 section 全是随机字节（`section 0 is corrupted!`）。
  解决办法：把**旧的 title.keys 备份**里的条目合并进现在的 title.keys
  （本机在 `Firmwares/title.keys.old-backup` 里就有这条：
  `01007ef00011e8000000000000000003 = 2566d59be1cb9bfcc2c1b8c648b8ed0d`）。
  合并后 NCA 里的 `.tik` 也能正常解析了。
- **hactool 不展开 `~`**：`--exefsdir=~/sw-work/exefs` 这种写法它不报错，
  但文件会**一个都存不下来**（日志里照样打 "Saving xxx to ~/..."）。
  一律写绝对路径。
- **更新 NCA 的 exefs 要分两步取**：更新包的 section 0 是个 PFS0，
  `--exefsdir` 对它不生效（一个文件都不存）。先存成文件再当 PFS0 解：

  ```bash
  hactool -t nca --section0=section0.bin update.nca
  hactool -t pfs0 --outdir=exefs/ section0.bin
  ```

  本体 NCA 可以一步到位：`hactool -t nca --exefsdir=exefs/ base.nca`。
- **合并 RomFS 可以一步做完**：`--basenca=` 加上 `--romfsdir=` 会直接解出
  合并后的文件树（41062 个文件），不用先落 `merged/romfs.bin` 再解一次：

  ```bash
  hactool -t nca -x -y --basenca=base.nca --romfsdir=extracted/romfs update.nca
  ```
- **Debian 13 的 Vulkan 头偏旧**（1.4.309，没有 `VK_KHR_unified_image_layouts`）：
  上游已经兼容，反馈回路会退回 `ATTACHMENT_FEEDBACK_LOOP_OPTIMAL` 布局，功能不受影响。
  （本机现在是 Fedora 45，Vulkan 头更新，这条只作参考。）
- **更新包的标题 ID = 本体 ID + 0x800**（王国之泪那边同理）。
- **游戏语言是区域语言**：`savagewind.ini` 里 `text_language = CNzh` / `TWzh` / `USen`……
  简体中文对应游戏自己的 `zh-Hans` / `zh-Hant` 两个代码，**不是** `zh-CN` / `zh-TW`——
  游戏自己的表里就是这两个，给错了游戏会退回日语。
- **全屏和分辨率**在 `savagewind.ini`（`fullscreen = 1` 等），
  也要能在游戏的 Options 里改（上游把设置页接进了游戏自己的菜单）。
- **Debian 13 的 Vulkan 头偏旧**（1.4.309，没有 `VK_KHR_unified_image_layouts`）：
  上游已经兼容，反馈回路会退回 `ATTACHMENT_FEEDBACK_LOOP_OPTIMAL` 布局，功能不受影响。
- **编译很慢**：游戏代码是一大堆 C，`make gen` 约 1 分钟，`make -j` 编译 12 核约 20 分钟。
- **需要 Vulkan 1.3 的驱动**（Mesa RADV / ANV / NVK 等）和 AVX2 的 CPU；
  没有 AVX2/FMA 时游戏代码会退成兼容模式（`-march=x86-64-v2`），慢一些。
- **上游的 issue 里 Linux 常见坑**：HDR 显示异常（#369）、
  菜单 UI 错位（#90）、Arch 上的杂七杂八（#7）；
  菜单卡顿（#688，0.5.0-alpha.5 的 RX 7600）、帧率上不去（#692）也有人在报。
- **官方不提供也不接受游戏文件的分享**，issue 里贴游戏文件、密钥、生成代码的评论会被删。

## 本机状态（2026-11-20 核实）

- Fedora 45 / amd64，12 核（AVX2+FMA，走 v3 构建），AMD Radeon RX 9070（RADV）。
- **用官方 AppImage 装**（`~/Applications/Savage_Wind-0.6.0-alpha.6-x86_64.AppImage`，
  上游 v0.6.0-alpha.6）——AppImage 自带 zig cc 和 swgen，**不用装编译工具链**，
  也不需要 .NET / Python 依赖（本机就没装 gcc / clang / make）。
  用无窗口模式做首次 setup：

  ```bash
  ~/Applications/Savage_Wind-<版本>-x86_64.AppImage --prepare --game <含 exefs/ 和 romfs/ 的目录>
  ```

  它自己拷贝游戏文件到 `~/.local/share/savagewind/extracted/` 并在本机编译游戏代码
  （本机 12 核 **195 秒**，产出 `swgame.so` 226M），最后打印「游戏文件已就绪」。
  首次运行日志里 `romfs: version "1.9.0", 41062 files, 15129626830 bytes` 就是校验通过。
- 游戏文件是本机自己解的 1.9.0（本体 XCI + 1.9.0 更新 NSP，hactool 合并 BKTR），
  加上卡带里的两个 DLC（aoc1 7094 文件、aoc2 1 文件，都是 3.0）。
  解出来的文件放在 `~/sw-work/extracted/`（`exefs/` `romfs/` `aoc1/` `aoc2/`）。
- 之前是从源码构建的（`~/Projects/SavageWindRecomp`，基于上游 v0.5.1-alpha.5），
  那份源码目录和 `~/.local/share/savagewind` 已经不在了，等于从零重装。
  上游自带的 Linux 测试脚本：`tools/linux/test_run.sh`，
  `SAVAGEWIND_TEST_WARP=plateau` 可以跳到初始台地、`SAVAGEWIND_INPUT` 模拟按键。
- 桌面入口：`~/.local/share/applications/savagewind.desktop`（右键还有
  「Savage Wind Configuration」），图标在 `~/.local/share/icons/hicolor/`。

# TotK Recomp（塞尔达传说：王国之泪 PC 静态重编译版）

- 仓库地址：https://github.com/valentderah/totk-pc-recomp
- 许可证：MIT
- 同名但**不同项目**的注意：[Chiuauaa/oot-pc-recomp](https://github.com/Chiuauaa/oot-pc-recomp)
  是**时之笛**（Ocarina of Time）的重编译版，和本项目（王国之泪）不是一回事，别搞混。

把《塞尔达传说：王国之泪》从 Switch 静态重编译成原生 Windows / Linux x86-64 程序，
配 Vulkan 渲染器。**release 里不含游戏数据，也不含任何从游戏派生的代码**：
安装器在玩家自己的机器上把游戏编译出来。

上游每两三天出一个新版本（2026-11 时已到 v0.3.13），
**每个版本都是完整的 Setup 安装器**（Linux 是 AppImage，Windows 是 zip）。

## Release 资产

| 资产 | 说明 |
| --- | --- |
| `TotK-Setup-x86_64-<版本>-linux.AppImage` | Linux amd64 安装器（自包含）|
| `TotK-Setup-<版本>-windows.zip` | Windows 安装器（就地解压）|

**注意资产命名在版本间变过**：v0.3.1 是 `TotK-Setup-0.3.1-x86_64.AppImage`（版本号在中间），
v0.3.13 改成了 `TotK-Setup-x86_64-0.3.13-linux.AppImage`（版本号在后面、多了 `-linux`）。
写脚本别把旧命名硬编码进去。

## 安装（按平台）

### Linux

AppImage 免安装（推荐，不需要 sudo）：

```bash
mkdir -p ~/Applications
gh release download <tag> --repo valentderah/totk-pc-recomp \
  --pattern 'TotK-Setup-x86_64-<版本>-linux.AppImage' --dir tmp/totk --clobber
chmod +x tmp/totk/TotK-Setup-x86_64-<版本>-linux.AppImage
```

跑起来是图形安装器，按提示选：

- **本体**（XCI 或 NSP）和**更新包**（NSP，1.1.0 / 1.2.1 / 1.4.3 三选一）；
- 自己的 `prod.keys`；
- 可选**固件目录**（解包后的固件的 `system/Contents/registered`，
  或模拟器的安装目录）——**中日韩繁中文字需要它**，否则游戏缺这些字形；
- 游戏语言和安装目录。

装完的目录里只有游戏本体，安装器和工具留在外面：

```
<安装目录>/
  totk, SDL3.so.0, libopus-0.so.0, totk_xlate     # Linux 上没有 .exe 后缀
  nvngx_dlss.dll          # NVIDIA DLSS，安装器的 DLSS 步骤下载
  totk.ini                # 语言等设置
  exefs/, romfs/          # 从玩家自己的文件里解出来的
  shared-fonts/           # Switch 系统字体（含中日韩繁中字形）
  profile/                # 存档
  mods/                   # 玩家的 mod
  logs/                   # totk.log 和前几次的 totk.1.log ~ totk.4.log
  shader-cache/           # shader 翻译缓存
  totk-install.json       # 这个目录是用哪个版本、哪些输入构建的
```

命令行装法（不跑图形界面）：

```bash
cd <Setup 目录>
python -m installer --base <本体> --update <更新包> --keys <prod.keys> --lang zh-Hans --dest <安装目录> [--firmware <固件目录>]
```

已有 `totk-install.json` 的目录上再跑时，**只需要 `--dest`**，
游戏文件只在需要重新解包时才要再给。

### macOS

上游**没有 macOS 构建**（只出 Linux 和 Windows）。

### Windows

下载 `TotK-Setup-<版本>-windows.zip` 解压后跑 `TotK-Setup.exe`，
装完跑安装目录里的 `totk.exe`。

## 首次安装要多久

首次安装是**真编译**：把游戏的 ExeFS 提升（lift）成 C，编译链接成 `totk.exe`，
再准备 shader。实测本机 12 核：

| 步骤 | 耗时 |
| --- | --- |
| 提取（解 ExeFS / RomFS，17GB）| 约 2 分钟 |
| 提升（ExeFS → C）| 约 30 秒 |
| 编译链接 `totk.exe`（24 核约 3 分钟）| 约 2.5 分钟 |
| 准备 shader（46671 个 shader）| 约 4 分钟 |

**峰值需要约 36 GiB 磁盘**，装完约 20 GiB。

## 更新

**更新不用重新编译整包，也不用重新解游戏文件。**
上游的固定说法是：

> To update, run the new installer on your existing install folder.
> It keeps your game files, saves and `totk.ini`, and rebuilds `totk.exe` and the shaders.

即：拿新版安装器跑在旧安装目录上，**游戏文件、存档、`totk.ini` 都保留**，
只重建 `totk.exe` 和 shader。

具体到步骤：

- **提取**：跳过（游戏文件没变，校验哈希一致）。
- **提升 + 编译**：会重跑，但**只重编变化的部分**——
  编译步骤的指纹包含 runtime 库（`libtotkrt.a`），所以**换了版本一定要重编**，
  这不是「从源码重编译整个项目」，是重新生成并链接游戏代码；
  实测重编约 2.5 分钟。
- **准备 shader**：runtime 变了就要重做，约 4 分钟（1.5GB 缓存）。
- **存档**：保留。1.2.1 和 1.4.3 的存档互转（1.4.3 加载时转换 1.2.1 的存档）。

更新步骤：

```bash
# 1. 下载新版安装器
gh release download <新tag> --repo valentderah/totk-pc-recomp \
  --pattern 'TotK-Setup-x86_64-<新版本>-linux.AppImage' --dir tmp/totk --clobber

# 2. 解出 Setup 目录（AppImage 里的安装器目录，后面 rebuild 模式要用）
chmod +x tmp/totk/TotK-Setup-x86_64-<新版本>-linux.AppImage
#    解压到 ~/Applications/TotK-Setup-<新版本>/

# 3. 在旧安装目录上跑新安装器（只需要 --dest）
cd ~/Applications/TotK-Setup-<新版本>
python -m installer --dest <游戏安装目录>
```

**新版安装器目录要留着**：`build-cache/rebuild.txt` 里记的 rebuild 命令
指向跑过的那一版 Setup 目录，mod 的 code cheat 重建需要它（`installer/gui.py --rebuild <安装目录>`）。
**Setup 目录删了或挪了，mod 重建就用不了了**，只能重跑一次安装。

图形界面下 v0.3.11 起会记住上次的路径（本体、更新包、keys、固件），
存在 `$XDG_STATE_HOME/totk-recomp/setup.json`（Windows 是 `%LOCALAPPDATA%\TotK Recomp\setup.json`），
**只存路径，不存 keys 文件内容**。

## 检测当前安装方式

```bash
cat <游戏安装目录>/totk-install.json | python3 -m json.tool | head -20
cat <游戏安装目录>/build-cache/rebuild.txt
ls ~/Applications/ | grep -i totk
```

判据：

- 安装目录里有 `totk-install.json` → **Setup 安装器装的**（这是唯一的官方分发方式）。
  其 `release` 字段是当前版本（如 `0.3.1+ab1dee5`），`inputs` 记着本体和更新包的 sha256。
- `build-cache/rebuild.txt` 里的路径指向当初那版 Setup 目录 → rebuild 依赖它。
- `~/Applications/TotK-Setup-x86_64.AppImage` 是当初下载的安装器 AppImage。

## 更新

**沿用当初的安装方式**：都是 Setup 安装器装法，更新就是「下新版 Setup → 跑在旧安装目录上」。

查最新版本：

```bash
gh api repos/valentderah/totk-pc-recomp/releases --jq '.[0] | "\(.tag_name)\t\(.published_at)"'
```

**更新前先备份存档**：

```bash
cp -a <游戏安装目录>/profile <游戏安装目录>/profile.bak
```

## 注意

- **本机状态（2026-11-09 核实）**：Debian 13 (trixie) / amd64（12 核，有 AVX2+FMA，
  走 **v3** 构建），游戏装在一个独立挂载点的 `Games/TotK-Recomp` 下（挂载点路径按本机情况替换）。
  **已从 v0.3.1（`0.3.1+ab1dee5`）更新到 v0.3.13（`0.3.13+ae30a61`）**，
  更新实测 486 秒，`totk --selftest` 0 failures 通过。
  Setup 目录 `~/Applications/TotK-Setup-0.3.13/`（**rebuild 依赖，不能删**），
  安装器 AppImage `~/Applications/TotK-Setup-x86_64-0.3.13-linux.AppImage`，
  安装日志 `~/Applications/totk-install.log`。
- **本次更新实测各步耗时**（12 核）：check 7 秒（校验 17GB romfs 哈希）、
  fonts 0 秒（`shared-fonts/` 已在，跳过）、**extract 跳过**（"current"）、
  lift 36 秒、**compile 170 秒**、**prepare 272 秒**（46671 个 shader）。
  也就是说更新只需重编 + 重做 shader，不必重新解 17GB 游戏文件。
- **更新前先备份了 `profile/` 和 `totk.ini`**（各留一份 `.bak-2026-11-09`），
  更新后确认存档文件 mtime/大小都没变、`totk.ini` 逐字节相同。
- **`build-cache/rebuild.txt` 里的路径要改成 Setup 目录的永久位置**：
  安装器写的是**当时运行安装器的那份解压目录**，直接从临时目录跑的话，
  临时目录清掉后 mod 的 code cheat 重建就废了。
  更新时把里面的路径从临时解压目录改成新的 `~/Applications/TotK-Setup-<版本>/`。
- **本体 XCI 和更新 NSP 的文件名都被截断过**：本体是
  `TOTK [0100F2C0115B6000][v0].xci`，更新包是
  `The Legend of Zelda Tears of the Kingdom v1.4.3 [0100F2C0115.nsp`
  （**结尾少了 `B6800]`**）。这两个名字是下载时改名过的，重装时按 `totk-install.json`
  里记的 sha256 对得上就行，别纠结文件名。
- **更新包的标题 ID = 本体 ID + 0x800**：本体 `0100F2C0115B6000`，
  更新包 `0100F2C0115B6800`。文件名尾部 `001` / `002` 才是 DLC，
  看到 `...6800` 别当成 DLC 装错。
- **XCI 的 update 分区是主机固件，不是游戏更新**——游戏更新要单独下 NSP。
- **旧密钥（≤0f）解不了 fw16+ 的 NCA**。本机用的是 v22.5.0 的 `prod.keys`
  （在 Switch 文件目录的 `Firmwares/` 下，旧版备份在同目录 `prod.keys.old-backup`），
  同步副本在 `~/.switch/prod.keys`。
- **中文字体需要固件**：不选固件目录安装器会警告然后继续，但游戏缺中日韩繁中字形。
  本机用的是固件 20.4.0。
- **keys 文件按 keygen 十六进制命名**（`_0a` = 10），不是十进制。
- **CPU 至少要 x86-64-v2（SSE4.2）**；有 AVX2+FMA 才用更快的 v3 构建。
  本机是 v3。GPU 要支持 Vulkan 1.3。
- **AMD GPU 可以加一行配置提帧率**（v0.3.13 release notes 提到）：

  ```ini
  [performance]
  vk_recorder = 1
  ```

  黑了或者报 `VK_ERROR_DEVICE_LOST` 就改回 `0`，并把 `logs\totk.log` 附到 issue 里。
  本机是 AMD Radeon RX 9070。
- **v0.3.7 起有三个默认值变了**：`replay_async`、`gpl`、`astc_bc`。
  更新后如果崩溃或帧率异常，先怀疑这三个。
- **帧率掉了或卡顿就把 `replay_async = 0` 写进 `totk.ini`**。
- **安装器会自己下载 NVIDIA DLSS**（`nvngx_dlss.dll`，版本 310.9.1，
  从 NVIDIA 自己的 DLSS SDK 仓库，NVIDIA RTX SDK 许可），
  只在 Windows + NVIDIA RTX 上做；没网也能装完（警告，下次再试）。
- **安装中断了可以续**：安装器读 `totk-install.json`，从上次完成的步骤接着来。
- **装完可以删掉安装器和源文件**（XCI / NSP / NSZ / XCZ），但**Setup 目录要留着**给 rebuild 用。
- **这个 port 只在 Windows 和 Linux 上**，macOS 没有构建。

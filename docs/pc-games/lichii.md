# Lichii

- 仓库地址：https://github.com/zortos293/Lichii
- 官网：https://lichii.zortos.me
- 许可证：GPL-3.0
- 兼容性报告：https://lichii.zortos.me/reports

实验性的开源 Nintendo Switch / Switch 2 模拟器，C++ 编写，构建在 Eglantine、Eden、yuzu、
Sudachi 之上（rebrand 保留了这些上游的数据目录与配置键，所以已有的存档和设置能直接接着用）。
不是任何官方项目的发布版，与 Nintendo 无关。

**上游每��都在出实验性 build**，release tag 是 `build-<编号>`，编号持续递增
（2026-11 时已到 125）。其中一部分标记为 prerelease，一部分标记为正式 release，
**两者都可能只是滚动构建**，不要假定「非 prerelease 就是稳定版」。

## Release 资产

每个 build 提供 12 个资产，命名规则：

```
lichii-linux-x86_64[-pgo].AppImage     # Linux amd64，PGO 版
lichii-linux-x86_64[-pgo].tar.gz        # Linux amd64，PGO 版
lichii-linux-aarch64[-pgo].AppImage     # Linux arm64
lichii-linux-aarch64[-pgo].tar.gz
lichii-macos-arm64.dmg
lichii-macos-arm64.zip
lichii-windows-x86_64[-pgo].zip
```

- **PGO 版（带 `-pgo`）是上游推荐下载的**。PGO = profile-guided optimization，
  同样的程序用训练过的 profile 编译，Linux 用的是 Eglantine 的 profile，
  Windows 用的是在 Lichii 上跑 Ocarina of Time 训练出来的 profile，
  所以对别的游戏收益小一些；Windows 版还额外用了 clang-cl + ThinLTO。
- Linux 的 **AppImage 自带依赖**（内含 Qt 6、libavcodec 等），要求 **glibc ≥ 2.39**
  （Ubuntu 24.04 / Fedora 40 / Debian 13 / Arch）。
- Linux 的 **tar.gz 用系统库**，所以机器上必须自己装好 Qt 6 和常规库。
- macOS 只有 arm64（Apple Silicon），要求 macOS 15+，**没做公证**：
  第一次要右键选「打开」，或 `xattr -dr com.apple.quarantine lichii.app`。
- Windows 是就地解压的 zip，解压后跑 `lichii.exe`；终端里可以用 `eden-cli.exe`。

**下载只认这个仓库的 releases 页面。** 项目明确说明没有别的下载站：
lichii.org 等地方提供的包不是本项目构建的，而且已经有人报告过会装进恶意程序
（[#545](https://github.com/zortos293/Lichii/issues/545)）。

**游戏、密钥（keys）和固件（firmware）都不包含在 release 里**，需要自己合法获取。

## 安装（按平台）

### Linux

AppImage 免安装（推荐，不需要 sudo）：

```bash
mkdir -p ~/apps/lichii
gh release download <tag> --repo zortos293/Lichii \
  --pattern 'lichii-linux-x86_64-pgo.AppImage' --dir tmp/lichii --clobber
mv tmp/lichii/lichii-linux-x86_64-pgo.AppImage ~/apps/lichii/lichii.AppImage
chmod +x ~/apps/lichii/lichii.AppImage
~/apps/lichii/lichii.AppImage
```

想要菜单入口：把 AppImage 里的 `.desktop` 装到 `~/.local/share/applications/`，
图标装到 `~/.local/share/icons/hicolor/<尺寸>/apps/io.github.zortos293.lichii.png`，
然后跑 `update-desktop-database ~/.local/share/applications/` 和
`gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor`。
`.desktop` 的 `Exec` 建议写 **AppImage 的绝对路径**，否则图形会话的 PATH 里
没有 `~/apps` 时菜单点了打不开。

要用 tar.gz 版（走系统库）：

```bash
gh release download <tag> --repo zortos293/Lichii \
  --pattern 'lichii-linux-x86_64-pgo.tar.gz' --dir tmp/lichii --clobber
tar -xzf tmp/lichii/lichii-linux-x86_64-pgo.tar.gz -C ~/apps/lichii
```

需要先装 Qt 6（`libqt6core6t64` 等）和常规图形库。

Flathub 上**没有**收录（`flatpak remote-ls flathub | grep -i lichii` 为空），
本项目约定优先 flatpak，但这个只能退回 AppImage / tar.gz。

### macOS

下载 `lichii-macos-arm64.dmg` 挂载后把 `lichii.app` 拖进 `/Applications`（或 `~/Applications`）。
没做公证，第一次打开要右键选「打开」，或：

```bash
xattr -dr com.apple.quarantine /Applications/lichii.app
```

命令行入口可以软链：`ln -s /Applications/lichii.app/Contents/MacOS/lichii ~/.local/bin/lichii`。

### Windows

下载 `lichii-windows-x86_64-pgo.zip` 解压后跑 `lichii.exe`（PGO 版推荐）。
需要命令行入口时用同目录下的 `eden-cli.exe`。

## 检测当前安装方式

```bash
command -v lichii                       # 命令在不在 PATH 里
ls ~/apps/lichii/                       # AppImage / tar.gz 装的位置
find ~/.local/share/applications -name '*lichii*'   # 有没有装菜单入口
```

判据：

- `~/apps/lichii/lichii.AppImage` 存在 → AppImage 免安装装法。
  更新就是换掉这个文件。
- `~/apps/lichii/` 下有解压出来的 `lichii` 等一堆文件 → tar.gz 装法。
- `/Applications/lichii.app`（macOS）→ dmg 拖拽装法。

## 更新

**沿用当初的安装方式**，不要换：

- AppImage 装的：下载新 build 的同名 AppImage 覆盖掉，然后 `chmod +x`。

  ```bash
  gh release download <新tag> --repo zortos293/Lichii \
    --pattern 'lichii-linux-x86_64-pgo.AppImage' --dir tmp/lichii --clobber
  mv tmp/lichii/lichii-linux-x86_64-pgo.AppImage ~/apps/lichii/lichii.AppImage
  chmod +x ~/apps/lichii/lichii.AppImage
  ```

  覆盖前先把旧的改名备份，出问题好回退。
- tar.gz 装的：下载新 build 的 tar.gz 解压覆盖 `~/apps/lichii/`。
- macOS dmg 装的：重新下载拖进 `/Applications`（覆盖旧的 app bundle）。
- Windows zip 装的：重新下载解压覆盖整个目录。

查最新 build：

```bash
gh api repos/zortos293/Lichii/releases --jq '.[0] | "\(.tag_name)\tprerelease=\(.prerelease)\t\(.published_at)"'
```

存档目录沿用 Eden / Eglantine 的老路径（rebrand 有意不迁移也不删除），
所以换了 build 不会丢存档和设置，但**测试实验性 build 前还是先备份存档**。

## 注意

- **本机状态（2026-11-10 核实）**：Debian 13 (trixie) / amd64，
  `~/apps/lichii/lichii.AppImage`（`lichii-linux-x86_64-pgo.AppImage`，
  build-130，sha256 `7ac03030c7a66b4b7455ee4f46426fc139e870071a6968bae4a645401b770d9b`），
  菜单入口 `~/.local/share/applications/io.github.zortos293.lichii.desktop`，
  图标装在 `~/.local/share/icons/hicolor/{128x128,256x256}/apps/`。
  显卡是 AMD Radeon RX 9070 (Navi 48, RDNA4)。
- **更新时必须认准 PGO 资产名，别下错成非 PGO 版**：
  同一个 build 同时有 `lichii-linux-x86_64.AppImage`（非 PGO，约 95 MB）和
  `lichii-linux-x86_64-pgo.AppImage`（PGO，约 140 MB），**文件名只差一个 `-pgo`**，
  下错了等于把 PGO 版降级成非 PGO 版。
  判据是文件大小：PGO 版明显更大（2026-11 实测 build-125 是 140 MB，build-130 是 142 MB，
  非 PGO 版是 95 MB）。更新后核对 `ls -l ~/apps/lichii/lichii.AppImage` 看大小对不对。
  菜单项的 `X-AppImage-Version=` 标记可以手动同步成本次 build 号。
- **AppImage 只打包了 128x128 和 256x256 两个尺寸的图标**，
  装菜单时手动拷这两个尺寸即可，别的尺寸缺省走 Qt 自带的。
- **build-116 在 AMD RX 6000/5000/Vega/Polaris（RNDA2 及更老）上会崩**
  （Ocarina of Time 在启动后编译 shader 时崩在 `amdvlk64.dll`），
  build-117 修好了（[#657](https://github.com/zortos293/Lichii/pull/657)），
  驱动把没有 matrix core 的卡也报成支持，#583 的快速路径因此只在 RDNA3 及更新上跑。
  **RX 7000/9000 和 NVIDIA 不受影响。**
- **Ocarina of Time 夜里有白色条纹/拖影**（Navi、萤火虫、烟雾）：
  关掉「Emulation → Configure → Graphics → Use NVIDIA DLSS in place of a game's DLSS」，
  这是 DLSS 引起的，修复还在测试。
- Splatoon Raiders 里角色偏暗、带绿调、帧率也比早期 build 低，正在排查。
- **存档路径沿用 Eden / Eglantine 的老目录**，rebrand 不会搬也不会删。
- 上游文档里有些地方还沿用 yuzu / Eglantine 的名字（build target 仍叫 `yuzu`，
  输出在 Windows 上是 `lichii.exe`），别被名字迷惑。
- 这个项目迭代极快，**兼容性随游戏、硬件、驱动变化**，上游不做任何通用帧率保证，
  具体游戏能不能跑建议查上面的社区兼容性报告。

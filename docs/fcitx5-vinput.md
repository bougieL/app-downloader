# fcitx5-vinput

Fcitx5 的离线中文语音输入插件：本地 sherpa-onnx / 云端 ASR，可选 LLM 改写。
官方仓库：<https://github.com/xifan2333/fcitx5-vinput>（GPL-3.0）

- 文档站：<https://xifan2333.github.io/fcitx5-vinput/>
- 模型/Provider 注册表仓库：<https://github.com/xifan2333/vinput-registry>

## 安装方式清单

| 方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 系统包管理器（deb / rpm / Arch pkg） | `dpkg -l \| grep fcitx5-vinput` 能查到 | 见「更新」一节，必须先检测是哪种 |
| Flatpak | `flatpak list \| grep Vinput` | `flatpak update org.fcitx.Fcitx5.Addon.Vinput` |
| 免安装 tar 包 | `~/.local/share/fcitx5/addon/fcitx5-vinput.so` 存在 | 重新下载对应资产覆盖 |
| Nix / AUR / COPR / PPA | 由对应包管理器安装 | 沿用该包管理器的更新命令 |

## 检测当前安装方式

```bash
dpkg -l | grep fcitx5-vinput          # deb
flatpak list | grep Vinput            # flatpak
ls ~/.local/share/fcitx5/addon/        # 免安装 tar 包
```

检测不出来问用户，不要猜，也不要擅自换方式重装。

## Release 资产命名规则

版本号形如 `v2.4.2`：

- Debian / Ubuntu：`fcitx5-vinput_<版本>-1_debian12_amd64.deb`、`..._ubuntu24.04_amd64.deb`（还有 arm64）
- lite 版（纯云端 ASR，不带本地 ONNX 运行时）：`fcitx5-vinput-lite_<版本>-1_debian12_amd64.deb`
- Arch：`fcitx5-vinput-<版本>-1-x86_64.pkg.tar.zst`
- Fedora：`fcitx5-vinput-<版本>-1.fc43.x86_64.rpm`
- 通用 Linux：`fcitx5-vinput-<版本>-1_linux_x86_64_bundled.tar.gz`（含模型，可直接解压用）
- Flatpak：`fcitx5-vinput.flatpak`、`fcitx5-vinput-lite.flatpak`

## 安装（Debian / Ubuntu）

```bash
sudo apt-get install -y ./fcitx5-vinput_*.deb
systemctl --user enable --now vinput-daemon.service
fcitx5 -r            # 或 systemctl --user restart app-fcitx5@autostart.service
```

安装后插件落在 `/usr/lib/x86_64-linux-gnu/fcitx5/fcitx5-vinput.so`，
守护进程单元是 `/usr/share/systemd/user/vinput-daemon.service`。

## 模型

本地模型不随包分发，需要单独下载（注册表：`vinput-registry/registry/models.json`）。

```bash
vinput model add model.sherpa-onnx.x-asr-960ms-streaming-zipformer-transducer-zh-en-punct-int8
vinput model use  model.sherpa-onnx.x-asr-960ms-streaming-zipformer-transducer-zh-en-punct-int8
vinput model list
```

中文可选模型（`vinput-registry/registry/models.json`）：

| 模型 ID | 大小 | 说明 |
| --- | --- | --- |
| `model.sherpa-onnx.x-asr-960ms-streaming-zipformer-transducer-zh-en-punct-int8` | ~134 MB | 中英，流式，带标点，支持热词（推荐做输入法） |
| `model.sherpa-onnx.x-asr-zipformer-transducer-zh-en-punct-int8` | ~136 MB | 同上，离线整段识别 |
| `model.sherpa-onnx.sense-voice-zh-en-ja-ko-yue-int8` | ~166 MB | 多语种，不支持热词 |
| `model.sherpa-onnx.streaming-zipformer-small-ctc-zh-int8` | ~21 MB | 纯中文小模型，速度快，质量一般 |

## 更新

先检测安装方式，再沿用：

```bash
# deb
sudo apt-get install -y ./<新版本>.deb
# flatpak
flatpak update org.fcitx.Fcitx5.Addon.Vinput
```

模型是独立下载的，升级插件**不会**自动更新模型，需要 `vinput model remove <id>` 后重新 `vinput model add <id>`。

## 注意

- Debian 13 (trixie) 官方只发布到 `debian12` 的 deb，实测可以直接安装：
  依赖只有 `fcitx5`，`apt-get install -f` 无需降级本机依赖。
- `fcitx5 -r` 会在当前终端前台跑起 fcitx5（`-r` = replace），
  终端一关输入法就没了。用 `systemctl --user restart app-fcitx5@autostart.service` 更稳
  （该单元由 `~/.config/autostart/fcitx5.desktop` 经 systemd-xdg-autostart-generator 生成）。
- 默认热键：`Alt_R` 点击切换录音 / 按住说话，`Control_R` 语音改写选中文本，
  `Shift_R` 打开命令面板（`/asr`、`/model`、`/scene`、`/proc`）。
- 模型下载走 GitHub Release，网络慢时体积大的模型要等很久。

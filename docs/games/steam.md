# Steam（Linux / Flatpak）

- 应用 ID：`com.valvesoftware.Steam`（Flathub，flatpak 版）
- 官网：https://store.steampowered.com
- 相关上游：[Wine](https://www.winehq.org)、[Proton](https://github.com/ValveSoftware/Proton)

本篇记录 Flatpak 版 Steam 在 Linux 上的音频链路，以及 2026-11 排查的「游戏内滋滋声（爆音）」问题。

## 音频链路

Flatpak 版 Steam 跑 Windows 游戏时，声音要经过 Wine → PipeWire → ALSA → HDMI 四层：

```
游戏 → winepulse.drv (Wine 的 PulseAudio 驱动) → pipewire-pulse 兼容层
     → PipeWire 核心 → ALSA (snd_hda_intel) → HDMI → 电视/显示器
```

**只有 Proton 游戏会经过 `winepulse.drv`。** 看视频、听歌走的是宿主原生 ALSA/PipeWire，不经过这一层。
这是判断「同一个硬件上，为什么只有游戏有声音问题」的关键分界线。

Wine 的音频驱动优先级写死在 `mmdevapi.dll` 里，字符串字面量是 `pulse,alsa,oss,coreaudio`——
即先试 `winepulse.drv`，加载失败才回退 `winealsa.drv`。

## 排查音频问题用到的命令

### 看当前 PipeWire 拓扑

```bash
wpctl status                      # 设备、sink、stream 列表
pw-dump | python3 -m json.tool    # 完整属性（含 node.latency、audio.channels）
```

`wpctl status` 的 `Streams:` 一段最直观，能直接看到流的名字和它连到了哪个设备：

```
 ├─ Streams:
      75. ALSA plug-in [wine64-preloader]
           86. output_FR       > Mi TV:playback_FR	[active]
           87. output_FL       > Mi TV:playback_FL	[active]
```

流名带 `ALSA plug-in` = 走 winealsa；带 PulseAudio 字样 = 走 winepulse。

### 看 xrun（欠载）计数

xrun 是数字音频里出现爆音的直接原因。`ERR` 列是进程启动以来的累计错误数：

```bash
pw-top -b -n 2 | grep -E '^R ' | grep hdmi
```

计数持续增长 = 正在爆音；不增长 = 链路健康。

### 看显卡时钟跳变

AMD 显卡在负载低时会反复切降频档，视频时钟跳变会牵动 HDMI 音频：

```bash
cat /sys/class/drm/card0/device/pp_dpm_sclk    # 带 * 的是当前档位
cat /sys/class/drm/card0/device/gpu_busy_percent
cat /sys/class/drm/card0/device/hwmon/hwmon*/power1_average
```

### 看 HDMI 设备报告的音频能力

电视通过 ELD（EDID Like Data）向主机报告它支持几声道、哪些采样率：

```bash
cat /proc/asound/card0/eld#0.3
```

输出里 `speakers [0x1] FL/FR` 表示电视只接受立体声；`sad0_rates` 列出支持的采样率。

### 录制数字音频做离线分析

确认「声音是数字信号里就有的，还是模拟环节引入的」，录一段出来看波形和频谱：

```bash
# 录游戏的输出（stream 名字见 wpctl status）
pw-record --target="<stream 名>" --rate 48000 --channels 12 --format s16 out.wav

# 录 sink 实际送到设备的信号，要带 .monitor 后缀
pw-record --target=alsa_output.pci-0000_03_00.1.hdmi-stereo-extra3.monitor \
          --rate 48000 --channels 2 --format s16 out.wav
```

**注意**：录 sink 不加 `.monitor` 会录到全零数据——那是 sink 的输入侧，不是输出侧。
判断数字音频是否干净，看三个指标：有无削波（`|x| >= 32760`）、有无相邻样本跳变（咔哒声）、
高频段能量占比是否异常抬升（嘶声）。

```bash
uv run --with numpy python - <<'EOF'
import numpy as np
raw = open('out.wav','rb').read()
off = raw.find(b'data') + 8
x = np.frombuffer(raw[off:], dtype='<i2').astype(np.float64)
print('削波样本:', (np.abs(x) >= 32760).sum())
dd = np.abs(np.diff(x))
idx = np.where(dd > np.percentile(dd, 99.99) * 4)[0]
print('咔哒声:', len(idx), '个, 时刻:', [round(i/48000, 3) for i in idx[:10]])
EOF
```

## 已知问题

### 游戏内滋滋声（爆音），Windows 下正常

**现象**：Proton 游戏内有「滋滋」电流声，看视频没有；同一个游戏在 Windows 下正常。
角色说话时最明显，放音乐时轻一些。电视是 HDMI 输出。

**为什么 Linux 才有**：Wine 的 `winepulse.drv` 有两个还没合入上游的 bug：

- [MR9840](https://gitlab.winehq.org/wine/wine/-/merge_requests/9840)
  `winepulse.drv: A potential solution to audio crackling`——排空 buffer 时不留余量，
  jitter 直接造成欠载。
- [MR10792](https://list.winehq.org/hyperkitty/list/wine-gitlab@list.winehq.org/message/6GQNR3VMBTLXYEP6CUKSODHCJRFI3QMS)
  `winepulse.drv: Don't advance read pointer past data not yet consumed by PA`——
  winepulse 用「假时钟」推进读指针，必然与 PulseAudio 实际消费进度错位，产生重叠失真。
- [MR337](https://www.winehq.org/pipermail/wine-devel/2022-July/222335.html)
  `winepulse: Return device-specific values for GetMixFormat`——针对「设备声道数少于
  应用请求声道数」的场景。UE5 游戏请求 12 声道（7.1.4）而 HDMI 设备只有 2 声道时命中这条。

Windows 走原生 DirectSound，没有 `winepulse.drv` 这一层，所以不会中招。

**解决办法**：让 Wine 跳过 `winepulse.drv`，回退到 `winealsa.drv`。

```bash
# 全局生效，所有 Proton 游戏都走 winealsa
flatpak override --user --env=WINEDLLOVERRIDES=winepulse.drv=d com.valvesoftware.Steam
```

改完**必须重启 Steam**（旧进程的环境变量不会变）。验证：

```bash
flatpak run --command=sh com.valvesoftware.Steam -c 'echo $WINEDLLOVERRIDES'
# 应输出：winepulse.drv=d
```

启动游戏后 `wpctl status` 里流的名称应变成 `ALSA plug-in [wine64-preloader]`，
而不是 PulseAudio 相关名字。

Proton 脚本用 `append_to_env_str(self.env, "WINEDLLOVERRIDES", s, ";")` 追加而非覆盖，
所以这个 override 不会被 Proton 冲掉。

**同时加大 PipeWire 兼容层缓冲**（缓解 jitter）：

```bash
mkdir -p ~/.config/pipewire/pipewire-pulse.conf.d
cat > ~/.config/pipewire/pipewire-pulse.conf.d/10-bigger-buffer.conf <<'EOF'
context.properties = {
    pulse.min.quantum  = 1024/48000
    pulse.min.req      = 1024/48000
    pulse.default.req  = 2048/48000
    pulse.default.frag = 4096/48000
    pulse.idle.timeout = 0
}
EOF
systemctl --user restart pipewire-pulse
```

确认生效（这些属性挂在 `pipewire-pulse` 这个 client 上，不在 core 上）：

```bash
pw-dump | python3 -c "
import json,sys
for o in json.load(sys.stdin):
    if o.get('type')=='PipeWire:Interface:Client':
        p=o['info']['props']
        if 'pulse.min.quantum' in p:
            print(p['pulse.min.quantum'], p['pulse.default.req'])
"
```

**回滚**：

```bash
flatpak override --user --unset-env=WINEDLLOVERRIDES com.valvesoftware.Steam
rm ~/.config/pipewire/pipewire-pulse.conf.d/10-bigger-buffer.conf
systemctl --user restart pipewire-pulse
```

### 备选方案：绕过 HDMI 走光纤

主板上有 S/PDIF 输出（PipeWire 里是 `Ryzen HD Audio Controller 数字立体声 (IEC958)`）。
光纤是电气隔离的，能一并解决共地环路问题：

```bash
wpctl set-default 34    # 34 = IEC958 sink 的 id，用 wpctl status 查
```

电视需有 S/PDIF 输入口且切到对应输入源。

### 备选方案：强制游戏输出立体声

游戏请求 12 声道而设备只有 2 声道是触发条件之一。UE5 游戏在「设置 → 音频」里把输出声道
数改为立体声，可省掉下混这一步。也可以装 GE-Proton 10-27+ 用 `WINEALSA_CHANNELS=2`。

## 注意

- `winealsa` 直接绑定 ALSA 设备，不走 PipeWire 的采样率转换。Flatpak 沙箱里有 `/dev/snd`，
  且 `/usr/share/alsa/alsa.conf.d/50-pipewire.conf` 提供了 ALSA→PipeWire 桥接，所以能正常出声。
- 排查「只有游戏有声音问题」时，先确认问题是否只出现在 Proton 游戏。视频/音乐正常，
  基本可以锁定在 `winepulse.drv` 这一层，不用再查 HDMI 线、接地、显卡啸叫。
- 数字音频干净（无削波、无跳变、无高频底噪）但仍有滋滋声，说明噪音在模拟环节或耳机/音箱侧，
  不在 PC 内部生成的数字信号里。
- `docs/` 里不要出现本机路径与私有地址；上文的路径都用 `~` 或占位符。

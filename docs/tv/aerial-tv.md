# Aerial TV（国产电视桌面启动器）

- 仓库地址：https://github.com/pengchujin/aerial-tv
- 官网：https://aerial.qust.me（install.sh 一键安装器与详细文档都在这里）
- 许可证：MIT

参照 Apple TV 做的 Android TV **桌面启动器**（不是屏保应用）：无广告、无推荐流，
应用网格 + 文件夹管理，手机扫码整理布局，控制中心，内置 Apple TV 114 段航拍屏保与壁纸。
支持小米、TCL/雷鸟、创维、索尼等国产电视和盒子、投影仪，Android TV 5.0+。

## 包名与变体

| 资产 | 包名 | 适用 |
| --- | --- | --- |
| `aerial-standard.apk` | `app.orchard.tv` | Android 5.0+，首选 |
| `aerial-guard.apk` | `app.orchard.tv` | standard 提示「不允许安装桌面类 App」时用（guard 守护版） |
| `aerial-lite.apk` | `app.orchard.tv.lite` | Android 4.0–4.4 老设备（测试版） |

versionName 与 release tag 对应（v1.86 → 1.86）。

## 安装与更新（adb）

```bash
gh release download v1.86 --repo pengchujin/aerial-tv --pattern 'aerial-standard.apk' --dir tmp/aerial-tv --clobber
adb connect <电视IP>:5555
adb install -r tmp/aerial-tv/aerial-standard.apk
# README 建议的收尾两步（幂等、无害）：
adb shell cmd notification allow_listener app.orchard.tv/app.orchard.tv.guard.RevivalListener
adb shell cmd package compile -m speed -f app.orchard.tv
```

查当前版本：`adb shell dumpsys package app.orchard.tv | grep versionName`。

## 注意

- 更新用 `install -r`，布局/设置数据保留；实测小米 Android 14（armeabi-v7a）1.57 → 1.86 正常（2026-11）。
- 创维报 `INSTALL_FAILED_USER_RESTRICTED`：先
  `adb shell settings put global verifier_verify_adb_installs 0` 再装。
- 只有「无线调试」开关的 Android 11+ 电视（如索尼 BRAVIA）没有 5555 明文端口：
  先 `adb pair <IP>:<配对端口>` 输电视上的 6 位配对码，再 `adb connect <IP>:<无线调试端口>`；
  连上后可跑一次 `adb tcpip 5555` 固化，以后直接连 5555。
- 还原原生桌面：电视上 设置 → 应用 → Aerial → 清除默认设置，按主页键选回原桌面；
  或直接 `adb uninstall app.orchard.tv`。
- 个别机型（创维小湃 P3 盒子、OPPO K9 等）要改包名才能装/接管 HOME，README 的
  设备兼容表里有逐机型笔记，装前先查。

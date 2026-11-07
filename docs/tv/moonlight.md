# Moonlight（Limelight，游戏串流客户端）

- 仓库地址：https://github.com/moonlight-stream/moonlight-android
- 官网：https://moonlight-stream.org
- 许可证：GPL-3.0

NVIDIA GameStream / Sunshine 的开源串流客户端，配对 PC 后在电视上串流游玩。
历史名 Limelight，包名沿用 `com.limelight`。

## 资产

- `app-nonRoot-release.apk`：普通版，首选
- `app-root-release.apk`：root 版
- 无按 ABI 分包，一个包通吃

## 更新（adb）

```bash
gh release download <tag> --repo moonlight-stream/moonlight-android --pattern 'app-nonRoot-release.apk' --dir tmp/moonlight --clobber
adb install -r tmp/moonlight/app-nonRoot-release.apk
```

tag 带 `v` 前缀（如 `v12.2`）。查当前版本：
`adb shell dumpsys package com.limelight | grep versionName`。

实测 2026-11：12.1 → 12.2 正常升级，配对数据保留。

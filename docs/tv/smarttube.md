# SmartTube（YouTube TV 客户端）

- 仓库地址：https://github.com/yuliskov/SmartTube
- 许可证：非典型开源协议（源码公开，禁止商用再分发）

Android TV 上的第三方 YouTube 客户端，无广告、支持 4K/HDR、遥控器适配。

## 包名与渠道

- stable 渠道包名：`org.smarttube.stable`（另有 beta 等渠道，包名不同，可共存）
- release 资产按 ABI 分包：`SmartTube_stable_<版本>_{arm64-v8a,armeabi-v7a,universal,x86}.apk`
- 本电视是 armeabi-v7a（32 位），选 `armeabi-v7a` 包，不要用 universal

## 更新（adb）

```bash
gh release download <tag> --repo yuliskov/SmartTube --pattern 'SmartTube_stable_<版本>_armeabi-v7a.apk' --dir tmp/smarttube --clobber
adb install -r tmp/smarttube/SmartTube_stable_<版本>_armeabi-v7a.apk
```

查当前版本：`adb shell dumpsys package org.smarttube.stable | grep versionName`。
tag 名不带 v 前缀且带渠道后缀（如 `32.71s`），versionName 是纯数字（`32.71`）。

实测 2026-11：30.82 → 32.71（armeabi-v7a）正常升级，数据保留。

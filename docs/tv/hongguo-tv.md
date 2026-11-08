# 红果短剧 TV 版（多个第三方客户端）

字节官方没有出过红果短剧 TV 版，GitHub 上有多个第三方实现。**电视上已装 3 个**
（小米 MiTV-MFTR0，Android 14，2026-11 实测）：

| 包名 | 仓库 | 版本 | 说明 |
| --- | --- | --- | --- |
| `rocks.aston.hongguotv` | [aston314/hongguo-tv-releases](https://github.com/aston314/hongguo-tv-releases) | 0.2.155 | 933★，最活跃，多端（TV/webOS/Mac/Win/Linux） |
| `com.hongguotv.nativeapp` | [jycao2/hongguoTV](https://github.com/jycao2/hongguoTV) | 0.28.0+50 | 112★，Kotlin 原生仅 1.9MB，支持 WebDAV/NAS 本地库与类 TVBox 自加源；release 发在 [hongguoTV-updates](https://github.com/jycao2/hongguoTV-updates) |
| `dev.miniact.nativeapp` | [xy6176/miniact-tv](https://github.com/xy6176/miniact-tv) | 1.2.9.2 | 无广告，遥控追剧/选集/续播，pre-release |

三个包名互不相同，可共存。

## aston314 版（首选，常规更新走这个）

## 资产与变体

| 资产 | 适用 |
| --- | --- |
| `hongguo-tv-<ver>.apk` | 常规版，首选 |
| `hongguo-tv-android5-<ver>.apk` | Android 5.x 老设备 |
| `hongguo-tv-mpv-<ver>.apk` | 内置 mpv 播放器版（体积约 2 倍） |

APK 是多 ABI 通用包（lib 下含 armeabi-v7a / arm64-v8a / x86_64），32 位电视不用挑。
versionName 与 release tag 对应（v0.2.155 → 0.2.155）。SHA256 校验文件为 `SHA256SUMS.txt`。

## 安装与更新（adb）

```bash
gh release download v0.2.155 --repo aston314/hongguo-tv-releases --pattern 'hongguo-tv-0.2.155.apk' --dir tmp/hongguo-tv --clobber
sha256sum -c tmp/hongguo-tv/SHA256SUMS.txt   # 只校验下载的那行
adb connect <电视IP>:5555
adb install -r tmp/hongguo-tv/hongguo-tv-0.2.155.apk
```

查当前版本：`adb shell dumpsys package rocks.aston.hongguotv | grep versionName`。

- 包名：`rocks.aston.hongguotv`（versionName 与 tag 对应，v0.2.155 → 0.2.155）
- 实测小米 MiTV-MFTR0（Android 14，armeabi-v7a）安装并启动正常（2026-11）。

## jycao2 / MiniAct 版（安装与更新）

```bash
gh release download kotlin-v0.28.0-build.50 --repo jycao2/hongguoTV-updates --pattern '*.apk' --dir tmp/hongguo-tv --clobber
sha256sum -c tmp/hongguo-tv/SHA256SUMS
gh release download v1.2.9.2 --repo xy6176/miniact-tv --dir tmp/hongguo-tv --clobber
adb install -r tmp/hongguo-tv/hongguotv-kotlin-<ver>-android8.apk
adb install -r tmp/hongguo-tv/MiniAct-<ver>.apk
```

- jycao2 版包名 `com.hongguotv.nativeapp`，apk 文件名带 `-android8`（minSdk 26），
  纯 Kotlin 无 native 库，任意 ABI 通用；release 附 `SHA256SUMS`。
- MiniAct 包名 `dev.miniact.nativeapp`，minSdk 19。
- 2026-11-08 首装 jycao2 版时报 `INSTALL_FAILED_UPDATE_INCOMPATIBLE`（签名不匹配）：
  电视上有 2026-10-10 的残留记录但包已不在已装列表，`adb uninstall com.hongguotv.nativeapp`
  清掉脏数据后重装即可。

## 备选仓库

- [fyworld/hongguoTV](https://github.com/fyworld/hongguoTV)（27★）：也是
  `com.hongguotv.nativeapp`（versionCode 10030），装上会**降级覆盖** jycao2 版，没装
- [flywingjie/hongguoTV](https://github.com/flywingjie/hongguoTV)（25★）：同包名 debug
  构建（versionCode 16，minSdk 21 更低），同上没装；只有老设备（Android 5–7）才需要
- [inspirai-store/HongguoTV-NoAd](https://github.com/inspirai-store/HongguoTV-NoAd)、
  [qq294716498/HongguoTV-Phone-TV-NoAds](https://github.com/qq294716498/HongguoTV-Phone-TV-NoAds)：
  无广体验版，无 release 资产，未实测
- 同包名的几家用 versionCode 分高下：jycao2 10250 > fyworld 10030 > flywingjie 16

## 注意

- 内容源走红果官方接口，账号/播放数据与手机版红果不互通，遇版权下架属正常。
- 第三方客户端，GitHub 不校验 APK，安装风险自担。

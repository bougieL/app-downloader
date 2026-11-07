# TVAPP（TV 盒子应用合集）

- 仓库地址：https://github.com/youhunwl/TVAPP
- 官网：https://app.iyouhun.com（帮助中心 https://app.iyouhun.com/help）
- 许可证：Apache-2.0

收集全网 Android TV 电视盒子应用的 APK 合集仓库（27k+ stars），涵盖影视点播、电视直播、K歌、
游戏、实用工具、教育学习、运动健身等分类。README 的「一览表」是总索引：每个 APP 标注版本、
下载直链和状态（🟢 可用 / 🔴 失效，🔴 的别下）。仓库同时还维护 TVBox / 影视仓等影音壳的
接口源清单和 IPTV 直播源清单（README 下方两节），接口本身免费，切勿付费购买。

## 分发方式

**不走 GitHub Release**（仓库没有 release），APK 直接放在仓库的分类目录里：

- `TVBox/`：空壳 TVBox，白壳/黑壳、java/python、32/64 位多个变体，进文件夹看版本说明
- `影视/`：影视仓、FongMi影视、OK影视、月光宝盒、宝盒TV、小苹果影视等
- `电视直播/`：我的电视、酷9直播、HTV直播、星火直播等
- `K歌/` `娱乐应用/` `实用工具/` `教育学习/` `运动健身/`：各自分类
- 少数 APK 在仓库根目录（如 MX Player 的三个变体）

下载用 README 一览表里的直链：

```bash
curl -L -o <输出>.apk "https://github.com/youhunwl/TVAPP/raw/refs/heads/main/<仓库内路径>"
```

- 直链都是 `raw/refs/heads/main/<路径>` 形式，等价于
  `https://raw.githubusercontent.com/youhunwl/TVAPP/main/<路径>`。
- 中文路径不编码也能直接下（实测 GitHub raw 接受 UTF-8 原样字节，curl 返回 200）；
  README 里部分链接是预编码过的，两种形式都能用。
- 一览表里部分「下载」链接指向**目录**而不是 APK 文件（如 TVBox、FongMi影视、酷9直播），
  要先列目录再挑具体版本：

  ```bash
  gh api repos/youhunwl/TVAPP/contents/<分类目录>/<应用目录> --jq '.[].name'
  ```

- Kodi 这类条目外链第三方镜像（阿里云），不在本仓库内。
- 官网 app.iyouhun.com 也能下载，另有详细的使用教程。

## 更新

这些 APK 装在电视/盒子上，不装在本机，没有本机更新流程。更新就是回仓库看对应 APP 的
版本和状态标记（README 一览表），或看官网。

## 包名对照（实测 2026-11）

给电视清理/更新时需要知道仓库 APK 对应的 Android 包名，README 一览表里没有：

- OK影视（`OK影视-电视版-*.apk`）→ `com.fongmi.android.tv`，**和 FongMi 影视同包名**，
  互相覆盖，装谁谁生效；「OK影视内置共存版」才是独立包名的共存版
- 月光宝盒Max → `tv.max.bh`（versionName 是 3.x 递增，和文件名里的日期版本是两套）
- 欧歌影视 v6.1.7 → `com.huawei.himovceif`；同设备另有 `com.huawei.himovceie`（6.2.8，按版本推断为魔改线，未实测）；
  旧包名 `com.fomi.anroid.tw`（label 就是「欧歌影视」）是历史版本，装了新包名后旧包名不会自动升级，
  会残留成两个应用
- SimpleLive TV 版 → `com.xycz.simple_live_tv`（仓库 1.7.9）；另有变体包 `com.xycz.simple_live_tv_app`

查未知包名属于哪个应用：`adb pull` 下 APK 后解析 `AndroidManifest.xml` 的字符串池
（包名 / versionName 都在里面）；README 里的应用对照本机名可以用 `dumpsys package <pkg>` 的
versionName + 仓库一览表版本号匹配。

## 注意

- 一览表里 🔴 状态的（OurTV、电视家修复版、小鸡模拟器TV版、贝乐虎儿歌、饭太硬等）已失效，别下。
- README 个别直链有笔误：MX Player 1.94.0 的链接少了 `main/` 这一段
  （`raw/refs/heads/MXPlayer-...apk` 会 404），手动补上分支名即可。
- 收录的 APK 多为第三方魔改/解锁版；仓库要求「标注来源 + 确认无恶意代码 + 测试截图」才收录，
  但 GitHub 不校验 APK、没有官方签名可核对，安装风险自担。
- 仓库定位「仅用于个人学习，严禁商用」。
- 接口源 / 直播源里的 GitHub raw 地址在国内使用需自行加代理（README 原话）。
- APK 装到电视/盒子用 U 盘侧载或 `adb install`，本仓库只管下载。

# X-plore（双栏文件管理器）

- 官网：https://www.lonelycatgames.com/apps/xplore
- 许可证：专有（付费解锁高级功能）

Lonely Cat Games 出品的双栏文件管理器（SMB/FTP/SFTP/WebDAV/云盘）。
**不走 GitHub**（无仓库、无 release），APK 只从官网 WordPress 下载管理器分发，
下载链接形如 `https://www.lonelycatgames.com/download/x-plore-<版本>-apk` 页面里的
`?wpdmdl=<id>&refresh=<token>` 直链，版本号看页面正文（如 4.46.00）。

## 更新（adb）

从官网页面拿直链下载后：

```bash
adb install -r Xplore-<版本>.apk
```

查当前版本：`adb shell dumpsys package com.lonelycatgames.Xplore | grep versionName`。

## 注意：签名坑（2026-11 实测）

- 官网 APK 的签名是 `META-INF/LCGKEY.RSA`；**第三方市场装的重签名版和官方版签名不匹配**，
  `adb install -r` 会报 `INSTALL_FAILED_UPDATE_INCOMPATIBLE: signatures do not match`。
- 换官方版必须先 `adb shell pm uninstall com.lonelycatgames.Xplore`（连数据一起清，
  `uninstall -k` 保数据也不行——签名不同的残留数据同样挡安装），再装官方版。
  X-plore 的数据只有收藏和服务器配置，清掉损失不大。
- 装完是官方签名，以后可直接 `install -r` 升级。

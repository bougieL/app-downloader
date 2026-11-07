# Tessera

- 仓库地址：https://github.com/bougieL/terminal-manager-releases

Electron 桌面终端管理器，通过 GitHub Release 对外发布安装包。

## 资产与安装（按平台）

### Linux

| 架构 | deb | AppImage |
| --- | --- | --- |
| x86_64 | `tessera_<版本>_amd64.deb` | `tessera-<版本>.AppImage` |

系统级安装（需要 sudo）：

```bash
sudo apt install ./tessera_<版本>_amd64.deb
```

不想用 sudo 就用 AppImage：`chmod +x` 后直接运行，或放 `~/apps/tessera/` 再软链到 `~/.local/bin/tessera`。

目前只发布 x86_64 资产，arm64 机器需要自己用源码构建或等待发布。

### macOS

资产：`tessera-<版本>.dmg`（挂载后拖 `Tessera.app` 进 `/Applications`）、`Tessera-<版本>-arm64-mac.zip`（免安装 `.app`，只有 arm64 版）。

macOS 端暂时只有 Apple Silicon（arm64）构建。

### Windows

资产：`tessera-<版本>-setup.exe`（安装器）。

## 忽略的文件

同一个 release 里还有 electron-updater 的元数据，别当安装包下载：

- `latest.yml` / `latest-linux.yml` / `latest-mac.yml`：自动更新索引
- `*.blockmap`：差分更新用
- `builder-debug.yml`：构建调试信息

## 注意

- 升级前先退出正在运行的 Tessera，否则安装器覆盖不了正在占用的文件。
- Electron 应用用 AppImage 运行时，文件本身要可执行（`chmod +x`）。
- `latest-linux.yml` 里的 sha512 只对 AppImage 有效，对 `.deb` 无效（记录的大小和哈希都对不上实际发布
  的 deb，估计是打包后重传过的）。校验 deb 别拿 yml 比，直接用 `dpkg-deb -I <包>` 看 Version 对不对。
- `tessera --version` 不会打印版本号，会直接拉起 GUI 主进程并挂住，要看版本用 `dpkg -l tessera`。
- **脚本/非交互终端里 `sudo apt install ./xxx.deb` 会刷一串 debconf front-end 警告**
  （`debconf: 无法初始化前端界面：Readline / Teletype ... 这个界面要求可控制的 tty`），
  最后回落到 `Noninteractive` 并正常装完。属正常现象，不影响结果，不必因此重跑。

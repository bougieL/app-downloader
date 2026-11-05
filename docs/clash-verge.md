# Clash Verge Rev

- 仓库地址：https://github.com/clash-verge-rev/clash-verge-rev
- 官网：https://www.clashverge.dev

基于 Tauri 的 Clash 图形客户端，支持 Windows、macOS、Linux。

## 资产与安装（按平台）

### Linux

| 架构 | deb | rpm |
| --- | --- | --- |
| x86_64 | `Clash.Verge_<版本>_amd64.deb` | `Clash.Verge-<版本>-1.x86_64.rpm` |
| aarch64 | `Clash.Verge_<版本>_arm64.deb` | `Clash.Verge-<版本>-1.aarch64.rpm` |

另有 `Clash.Verge_<版本>_armhf.deb`（32 位 ARM，别选错）。

系统级安装（需要 sudo）：

```bash
sudo apt install ./Clash.Verge_<版本>_amd64.deb
```

装在 `/usr/bin/clash-verge`。服务模式（`clash-verge-service`，TUN 模式要用的 systemd 服务）由包里的
`/usr/bin/clash-verge-service-install` 脚本安装，**deb 升级不会自动保留它**，见下面「升级后 TUN / 服务模式失效」。

#### 升级后 TUN / 服务模式失效

- **现象**：deb 升级完，系统代理/TUN 全下线：`mixed-port` 不再监听、`ip link` 里没有 `Mihomo` 设备，
  `journalctl -u clash-verge-service` 里出现
  `failed to open core IPC directory "/run/clash-verge-service/users/<uid>": No such file or directory`。
- **原因**：deb 的 prerm 会执行 `/usr/bin/clash-verge-service-uninstall`（卸载服务单元、内核和
  `/run/clash-verge-service/users/<uid>` 运行时目录），而 postinst 只 chmod 安装脚本，**不会重装服务模式**。
- **解决**：升级完手动重装服务模式，再退出并重开一次 Clash Verge GUI（GUI 启动时会经服务把内核按原
  配置拉起来，TUN 和 mixed-port 随之恢复）：

  ```bash
  sudo /usr/bin/clash-verge-service-install
  # 然后退出并重开 Clash Verge
  ```

  只 `systemctl restart clash-verge-service` 不够：服务的停止流程会清掉运行时目录，启动时的自动恢复
  仍会报目录缺失；手工把目录 `mkdir` 回来也会被下一次服务重启清掉。重启 GUI 是最稳的。

### macOS

资产：`Clash.Verge_<版本>_x64.dmg`、`Clash.Verge_<版本>_aarch64.dmg`，以及免安装的 `Clash.Verge_<版本>_x64.app.tar.gz` / `_aarch64.app.tar.gz`。

dmg 挂载后把 `Clash Verge.app` 拖进 `/Applications`；`.app.tar.gz` 解压出 `.app` 同样拖进 `/Applications`。

### Windows

资产：`Clash.Verge_<版本>_x64-setup.exe`（x86_64）、`_arm64-setup.exe`（ARM64），以及 `_x64_fixed_webview2-setup.exe` / `_arm64_fixed_webview2-setup.exe`（自带 WebView2 运行时，装在没预装 WebView2 的系统上用）。

## 注意

- 升级前先退出正在运行的 Clash Verge Rev，否则托盘进程会占着旧文件。
- `Clash Verge Rev` 的 macOS 版在部分系统上需要辅助功能权限才能改系统代理。

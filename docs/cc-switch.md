# CC Switch

- 仓库地址：https://github.com/farion1231/cc-switch
- 官网：https://ccswitch.io

跨平台桌面客户端，用来切换 Claude Code、Codex、OpenCode、DeepSeek 等工具的 API 服务商配置。

## 资产与安装（按平台）

### Linux

| 架构 | deb | rpm | AppImage |
| --- | --- | --- | --- |
| x86_64 | `CC-Switch-v<版本>-Linux-x86_64.deb` | `CC-Switch-v<版本>-Linux-x86_64.rpm` | `CC-Switch-v<版本>-Linux-x86_64.AppImage` |
| aarch64 | `CC-Switch-v<版本>-Linux-arm64.deb` | `CC-Switch-v<版本>-Linux-arm64.rpm` | `CC-Switch-v<版本>-Linux-arm64.AppImage` |

系统级安装（需要 sudo）：

```bash
sudo apt install ./CC-Switch-v<版本>-Linux-x86_64.deb
```

装在 `/usr/bin/cc-switch`。

不想用 sudo 就用 AppImage：`chmod +x` 后直接运行，或放 `~/apps/` 再软链到 `~/.local/bin/cc-switch`。

### macOS

资产：`CC-Switch-v<版本>-macOS.dmg`（挂载后拖 `CC Switch.app` 进 `/Applications`）、`CC-Switch-v<版本>-macOS.tar.gz`、`CC-Switch-v<版本>-macOS.zip`（都是免安装 `.app`）。

### Windows

资产：`CC-Switch-v<版本>-Windows.msi`（安装器）、`CC-Switch-v<版本>-Windows-Portable.zip`（免安装）、另有 `Windows-arm64.msi` 和 `Windows-arm64-Portable.zip` 给 ARM64 用。

## 注意

- 切换服务商配置会直接改写各工具的配置文件（`~/.claude/settings.json` 等），升级前确认没有未提交的重要改动。
- 升级前先退出正在运行的 CC Switch。

- **`gh release download` 偶尔会下出截断的 deb**：`dpkg-deb -I <包>` 还能正常读出 Version，
  但 `apt install` 报「错误： 归档文件中成员文件头无效」。校验要连数据段一起验：

  ```bash
  dpkg-deb -c <包>.deb > /dev/null && echo OK
  ```
- **脚本/非交互终端里 `sudo apt install ./xxx.deb` 会刷一串 debconf front-end 警告**
  （`debconf: 无法初始化前端界面：Readline / Teletype ... 这个界面要求可控制的 tty`），
  最后回落到 `Noninteractive` 并正常装完。属正常现象，不影响结果，不必因此重跑；
  装完用 `dpkg -l cc-switch` 核对版本即可。
- **本机状态（2026-11-12 核实）**：Debian 13 (trixie) / amd64，
  `dpkg -l cc-switch` 显示 4.0.30（从 4.0.28 升级），装在 `/usr/bin/cc-switch`，
  桌面项 `/usr/share/applications/CC Switch.desktop`。
  升级前需要 `pkill -x cc-switch` 关掉运行中的实例，否则 deb 覆盖不了被占用的文件。

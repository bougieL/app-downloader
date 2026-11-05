# Zed

- 仓库地址：https://github.com/zed-industries/zed
- 官网：https://zed.dev

高性能多人协作代码编辑器（Rust 编写），支持 Linux、macOS、Windows。Release 里提供 `.tar.gz`（Linux）、`.dmg`（macOS）、`.exe`（Windows），以及单独分发的 `zed-remote-server-*` 远程服务端。

用户配置在 `~/.config/zed/`，升级时不动。

## 资产与安装（按平台）

### Linux

资产：`zed-linux-x86_64.tar.gz`、`zed-linux-aarch64.tar.gz`。解压出 `zed.app/` 目录：

```bash
tar -xzf zed-linux-x86_64.tar.gz
mv zed.app ~/.local/
ln -sf ~/.local/zed.app/bin/zed ~/.local/bin/zed
```

图标：`cp -r ~/.local/zed.app/share/icons/hicolor/* ~/.local/share/icons/hicolor/`

桌面入口：复制 `~/.local/zed.app/share/applications/dev.zed.Zed.desktop` 到 `~/.local/share/applications/`，然后跑 `update-desktop-database ~/.local/share/applications/`。

**坑**：release 里自带的 `.desktop` 用的是 `Exec=zed` / `Icon=zed`。`Exec` 依赖桌面环境的 PATH 里有 `~/.local/bin`（通常要重新登录才有），`Icon=zed` 依赖图标主题缓存里有 `zed.png`。直接用自带的那份会出现菜单图标空白、点了打不开。装的时候把 `Exec`、`TryExec` 改成 `~/.local/zed.app/bin/zed` 绝对路径，`Icon` 改成 `~/.local/zed.app/share/icons/hicolor/512x512/apps/zed.png` 绝对路径，再 `gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor`。

远程服务端：资产 `zed-remote-server-linux-x86_64.gz`，解压后放 `~/.local/bin/zed-remote-server`。

### macOS

资产：`Zed-x86_64.dmg`、`Zed-aarch64.dmg`。挂载 dmg，把 `Zed.app` 拖进 `/Applications`。

命令行入口是 app 内的 `Zed.app/Contents/MacOS/zed`，需要单独软链到 `~/.local/bin/zed` 才能在终端用。

### Windows

资产：`Zed-x86_64.exe`、`Zed-aarch64.exe`（安装器）。安装后 `Zed.exe` 所在目录加入 PATH 才能在终端用 `zed`。

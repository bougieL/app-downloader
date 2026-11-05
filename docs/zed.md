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

## 更新

Linux 上是「整目录替换」：下载新的 `zed-linux-x86_64.tar.gz`，解压出 `zed.app/`，把旧的
`~/.local/zed.app` 挪走，再把新目录放回原位（`~/.local/bin/zed` 是指向
`~/.local/zed.app/bin/zed` 的软链，路径不变就不用重连）。换完再补两件事：

```bash
cp -r ~/.local/zed.app/share/icons/hicolor/* ~/.local/share/icons/hicolor/
gtk-update-icon-cache -f -t ~/.local/share/icons/hicolor
update-desktop-database ~/.local/share/applications/
```

桌面入口 `dev.zed.Zed.desktop` 是之前手动改过 `Exec`/`Icon` 绝对路径的那份，**不会被新版本覆盖**
（它拷在 `~/.local/share/applications/` 下），但新版本的 `.app` 里那份 `.desktop` 可能又改回相对路径，
要同步就重新按上面「坑」那段改一遍。

仓库同时发 `vX.Y.Z-pre` 预发布，稳定版用 `releases/latest` 取（`releases` 列表里 pre 排在前面，
别直接拿列表第一个）。远程服务端 `zed-remote-server` 是独立资产、单独更新，和编辑器版本不必一致。

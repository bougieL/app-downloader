---
name: download-app
description: 从 GitHub Release 下载开源应用并安装到本地。当用户要求下载、安装某个开源应用（Clash Verge Rev、CC Switch 等桌面客户端，或 ripgrep、fd 等命令行工具）时使用。
---

# 从 GitHub 下载应用

用 `gh` + `curl` 完成下载，**不要**额外写下载脚本。

## 1. 找仓库

- 先读 `docs/<应用名>.md`（每个应用一个文件，含仓库地址、资产命名规则、安装方式）。
- 没有对应文档时：用 `gh search repos "<关键词>" --limit 5 --json fullName,stargazersCount,description` 找官方仓库（取 star 最高、描述对得上的那个），然后**新建 `docs/<应用名>.md`**，写清仓库地址和应用介绍即可。

## 2. 查 Release 与资产

```bash
gh api repos/<owner>/<repo>/releases/latest --jq '.tag_name, (.assets[].name)'
```

需要指定版本用 `gh api repos/<owner>/<repo>/releases/tags/<tag>`。

## 3. 选对资产

以 `uname -s` / `uname -m` 判断平台与架构，再套用下面的排除规则。

架构关键词：`x86_64` `amd64` `x64` / `aarch64` `arm64` / `i386` `i686` `386` `32bit` `armv5-8` `armhf` `armel` `ppc64` `mips` `riscv` `s390x`。

必须排除：

- 非目标系统的资产：Linux/macOS 下排除 `windows` `freebsd` `openbsd` `android`；macOS 下排除 Linux 专用包。
- 非目标架构的资产（注意 `starship-arm-unknown-...`、`trivy_..._Linux-ARM.tar.gz` 这类 32 位 ARM 别选错）。
- 签名与校验文件：`.sig` `.sigstore` `.sha256` `.sha256sum`；`checksums.txt` / `SHA2-256SUMS` 单独留着做校验。

优先级：命令行工具 `.tar.gz` > `.zip` > 裸二进制；桌面客户端按平台选安装包（Linux 用 `.deb`/`.rpm`/`.AppImage`，macOS 用 `.dmg`/`.app.tar.gz`，Windows 用 `.exe`/`.msi`），注意同平台可能有多个架构变体（如 `amd64` 与 `arm64`），按目标架构挑。同名多个时 Linux 优先 `musl`（静态、依赖少）。

## 4. 下载与解压

临时文件放项目 `tmp/` 目录：

```bash
mkdir -p tmp/<app>
gh release download <tag> --repo <owner>/<repo> --pattern '<资产名>' --dir tmp/<app> --clobber
tar -xzf <资产名>        # .zip 用 unzip；.deb 用 dpkg-deb -x；.AppImage 直接 chmod +x 运行
```

## 5. 校验

release 提供 `checksums.txt` / `SHA2-256SUMS` 时用 `sha256sum -c` 校验，不匹配就停下报告。

## 6. 安装

先按 `uname -s` 判断平台，再按下面的对应方式装；不要把某一平台的路径写死。

**Linux**

- 命令行工具：可执行文件放 `~/.local/bin/<名字>`，`chmod +x`；仓库里附带的 man/completion 放 `~/.local/share/<app>/`。
- 桌面客户端（deb/rpm/AppImage）：需要系统级安装时用 `sudo apt install ./<包名>.deb`（rpm 用对应发行版的包管理器），装在 `/usr/bin`；不想用 sudo 就把整包放 `~/apps/<app>/`，AppImage `chmod +x` 后运行。
- 桌面入口（`.desktop`）：`Exec` / `Icon` 尽量写绝对路径，否则图形会话的 PATH 里没有 `~/.local/bin` 时菜单点了打不开、图标也可能空白；改完跑 `update-desktop-database ~/.local/share/applications/`。

**macOS**

- 命令行工具：可执行文件放 `~/.local/bin/<名字>`（同样是 `export PATH="$HOME/.local/bin:$PATH"`），并在 `~/.zshrc` 里固化。
- 桌面客户端（dmg / .app.tar.gz / zip）：`.app` 拖进 `/Applications`（或 `~/Applications`）；免安装版解压出 `.app` 后同样拖进去。需要命令行入口时把 `.app/Contents/MacOS/<名字>` 软链到 `~/.local/bin/<名字>`。
- 系统级安装（拷到 `/Applications`、改系统代理等）先跟用户确认。

**Windows**

- 命令行工具：可执行文件放 PATH 目录（如 `%LOCALAPPDATA%\Programs`），并加进用户 PATH。
- 桌面客户端：`.msi` 走安装器，`.exe` 走安装器，Portable 的 `.zip` 解压到 `~/apps/<app>/`。

安装到系统目录需要 sudo，先说明用途并征得同意。
结束时提示 PATH：`export PATH="$HOME/.local/bin:$PATH"`（Linux/macOS）。

## 输出

简短说明：应用名、版本、下载的资产名、安装位置。

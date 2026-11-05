---
name: download-app
description: 安装或更新开源应用与命令行工具。桌面客户端从 GitHub Release 下载安装（Clash Verge Rev、CC Switch 等）；CLI 工具（opencode、pi、codex、claude 等 coding agent）按 docs/ 里记录的安装方式处理，更新时先检测原安装方式并沿用它。
---

# 从 GitHub 下载应用

用 `gh` + `curl` 完成下载，**不要**额外写下载脚本。

临时文件放项目 `tmp/` 目录。

## 1. 找仓库

- 先读 `docs/<应用名>.md`（每个应用一个文件，含仓库地址、资产命名规则、安装方式）。
- 没有对应文档时：用 `gh search repos "<关键词>" --limit 5 --json fullName,stargazersCount,description` 找官方仓库（取 star 最高、描述对得上的那个），然后**新建 `docs/<应用名>.md`**，写清仓库地址和应用介绍即可。
- 少数项目不走 GitHub Release（自带分发站或 brew cask / 安装脚本），以文档里写的分发方式为准，别硬套 release 资产。

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

# 命令行工具（CLI）

coding agent 这类 CLI（opencode、pi、codex、claude 等）**不走 GitHub Release 下载**，各有各的安装方式
（安装脚本 / npm / pnpm / bun / brew / winget / 系统包管理器…）。

## 1. 找文档

先读 `docs/<应用名>.md`（opencode → `docs/opencode.md`，pi → `docs/pi.md`，codex → `docs/codex.md`，
claude → `docs/claude.md`，herdr → `docs/herdr.md`）。文档里应该有「安装方式清单」「检测当前安装方式」「更新」三节。
没有就照官方文档补一份。

## 2. 先检测，再动手

**已经装过的工具，更新前必须先检测出当初是怎么装的，并且沿用它。**

```bash
command -v <命令>            # 在不在 PATH 里
readlink -f "$(command -v <命令>)"   # 真实路径（命令常是软链或包管理器 shim）
```

判断方式：

- **先查各包管理器的全局列表确认是哪个**（最可靠）：npm / pnpm / bun / yarn / brew / winget / dpkg / rpm。
- 再看真实路径的**路径片段**（不要看完整目录，默认位置常被环境变量或配置改掉）：
  比如含 `packages/standalone/` 是 codex 的 standalone 安装，含 `node_modules/@openai/codex/` 是 npm 系。
- npm / pnpm 装出来的命令在 PATH 上是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
  这种情况下路径片段判据无效，以全局列表为准；也可以从 shim 里挖真实路径：

  ```bash
  grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v <命令>)")"
  ```

- 多个都命中说明装了多份，用 `which -a <命令>` 看 PATH 上生效的是哪个。
- 判断不出来就先问用户当初是怎么装的，不要猜。
- 包管理器本身如果由 mise / asdf 管着，用 `mise exec -- <包管理器> ...` 执行更新，
  免得更新到别的 node 环境里去。

## 3. 安装

没装过的话，按文档「安装方式清单」里的方式装，并记下选的是哪种。

## 4. 更新

**必须沿用第 2 步检测出来的安装方式，不要换方式。** 换方式的后果：同一个命令名出现两份、
PATH 上互相覆盖，或者升级命令去改了另一个包管理器的目录。

优先用工具自带的更新命令（`opencode upgrade`、`codex update`、`claude update`、`pi update`）——
它们内部就是按原方式更新。但它们的自动检测可能和实际情况不符，要和第 2 步的检测结果核对：

- 自带命令检测失败，或它用的方式和检测结果不一致 → 按文档「安装方式清单」里的
  「沿用该方式的更新命令」手动执行对应那一条。
- 检测结果里有多个候选 → 问用户，别擅自挑一个。

## 5. 写文档时的路径规矩

- 不要写死本机绝对路径：用 `~` 开头，或用占位符（如 `<codex 家目录>`）。
- 判定安装方式用**路径片段**，不要写死完整目录（如 `~/.codex/packages/standalone/`），
  因为默认位置可能通过 `CODEX_HOME`、`PI_CODING_AGENT_DIR` 这类环境变量改过。
- 文档里出现的具体路径片段要注明它是哪个工具自己约定的，别让人以为是系统标准位置。

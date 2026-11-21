# mise

- 命令：`mise`
- 仓库地址：https://github.com/jdx/mise
- 官网：https://mise.jdx.dev

开源的开发环境 / 运行时版本管理工具。用一套配置管理 node、python、rust、go、java 等
工具链的版本，也能装命令行工具（aqua registry）。

## 安装方式清单

mise 是「用来装别的东西的」工具，本身很少用系统包管理器装。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方安装脚本 `curl https://mise.run \| sh` | 真实路径是 `~/.local/bin/mise`（脚本默认装这里） | `mise self-update`，或重新跑安装脚本 |
| 手动下载 Release 二进制 | 同上 | `mise self-update` |
| Homebrew | 真实路径含 `/Cellar/mise` | `brew upgrade mise`（此时 `mise self-update` 会被 mise 拒绝，提示改用 brew） |
| npm / pnpm / bun | 包管理器的全局列表里有 `mise` | `npm install -g mise@latest`（pnpm / bun 同理） |
| 第三方 RPM / DEB 仓库 | `rpm -q mise` / `dpkg -l mise` 有输出 | 用装它的那个仓库更新（如 `dnf upgrade mise`） |

注意：

- 官方 Release **没有** `.rpm` / `.deb` 资产，只有裸二进制和 tarball；想用系统包管理器装得自己补仓库。
- Fedora 官方仓库里**没有** mise 包，`dnf install mise` 会失败；Flathub 上也没有 mise。
- 用官方安装脚本时脚本会自动改 shell 配置；手动放二进制的话激活要自己加
  （见下面的「激活」）。

## 检测当前安装方式

```bash
command -v mise
readlink -f "$(command -v mise)"
```

按**路径片段**对照上表（用片段而不是完整目录，默认位置可以被环境变量或配置改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `/Cellar/mise` | brew |
| `node_modules/mise/` | npm / pnpm / bun 系（PATH 上是 shim 脚本时，以全局列表为准） |
| `/.local/bin/mise` | 安装脚本或手动下载的二进制 |

前两种用 `mise self-update` 会被 mise 主动拒绝（提示「用包管理器更新」），这种情况按上表
对应那一条手动更新。

## 更新

**沿用检测出来的安装方式，不要换方式。**

自己下载的二进制（安装脚本 / 手动下载）直接：

```bash
mise self-update
```

`self-update` 会从 GitHub Releases 取最新的 stable 版本并校验签名后替换当前二进制。
默认跳过发布时间不足 24h 的版本（`self_update.minimum_release_age`），加版本号可以指定：

```bash
mise self-update v2026.11.9     # 指定版本
mise self-update --force         # 已经是最新也强制重装
```

mise 管理的**其它**工具是另一回事，用：

```bash
mise upgrade          # 升级所有已安装工具
mise upgrade node     # 只升级 node
```

## 激活

`mise` 本身装好不等于生效，需要在 shell 配置里激活，否则 `mise use` 设的工具不会进 PATH。

bash（`~/.bashrc`）：

```bash
command -v mise >/dev/null 2>&1 && eval "$(mise activate bash)"
```

zsh（`~/.zshrc`）：

```zsh
command -v mise >/dev/null 2>&1 && eval "$(mise activate zsh)"
```

用 `mise activate`（而不是 `mise activate bash`）可以让它自动识别当前 shell。

验证：

```bash
mise doctor | grep -E "activated|shims_on_path"
```

`activated: yes` 表示激活成功；`shims_on_path: yes` 表示 shim 目录已在 PATH 上。

## 常用命令

```bash
mise ls                 # 已安装的工具
mise ls-remote node     # 看远端可安装的版本
mise install node@22    # 装 node 22
mise use -g node@22     # 设成全局默认
mise use --node 22      # 在当前目录写一份 .mise.toml
mise exec -- node -v    # 用 mise 里的 node 跑一条命令
mise ls --current       # 当前目录生效的工具
```

`mise use -g <工具>@<版本>` 里版本可以只写大版本（如 `node@24`），
写进 `~/.config/mise/config.toml` 的是 `node = "24"` 而不是具体版本号，
之后 `mise upgrade` 会在这个大版本内自动跟进最新 patch。
想固定死版本就写全（`node = "24.22.3"`），`mise upgrade` 不会再动它。

## zsh 支持

mise 自带 zsh 补全：

```bash
mise completion zsh > ~/.local/share/zsh/site-functions/_mise
```

本机把 `~/.local/share/zsh/site-functions` 加进了 `fpath`，补全文件由
`~/.oh-my-zsh/custom/conf/project-cli.zsh` 自动生成和维护，不用手动跑。
详见 `docs/oh-my-zsh.md`。

另外 oh-my-zsh 自带 `mise` 插件，会自动生成 mise 的补全缓存，
在 `~/.zshrc` 的 `plugins=(...)` 里加 `mise` 即可。

## 注意

- `~/.local/bin` 在新装的系统上可能根本不存在，`install -m 755 mise ~/.local/bin/mise` 会报
  「没有那个文件或目录」，先 `mkdir -p ~/.local/bin`。
- mise 的 state 目录跟随 `XDG_STATE_HOME`。某些桌面应用会把这个变量指到自己的目录
  （例如 `~/.config/<应用名>`），这时 `mise doctor` 显示的 state 路径会跑到奇怪的位置，
  普通终端里应该是 `~/.local/state/mise`。确认方法：`env | grep XDG_STATE_HOME`。
- 官方安装脚本（`https://mise.run`）等价于「下载 Release 二进制 + 写 shell 配置」。
  想完全可控就手动下载并自己校验 `SHASUMS256.txt`：

  ```bash
  curl -sSL -O https://github.com/jdx/mise/releases/download/<版本>/mise-<版本>-linux-x64.tar.gz
  curl -sSL -O https://github.com/jdx/mise/releases/download/<版本>/SHASUMS256.txt
  sha256sum --ignore-missing -c SHASUMS256.txt
  tar -xzf mise-<版本>-linux-x64.tar.gz
  install -m 755 mise/bin/mise ~/.local/bin/mise
  ```

  资产名里的架构：`x64` = x86_64，`arm64` = aarch64，`armv7` = 32 位 ARM；
  同名再带 `-musl` 的是静态链接版本，依赖少的机器上优先选它。

- Fedora 上用 `dnf` 装别的东西时，如果 mise 里有同名工具，`mise exec -- <命令>` 可以在
  不改全局 PATH 的情况下临时用 mise 里那份。

- **官方安装脚本不会自己写 shell 配置**（2026-11 实测）：
  脚本装完只打印一行提示，让你自己把 `eval "$(mise activate bash)"` 写进 `~/.bashrc`。
  zsh 也要单独写 `eval "$(mise activate zsh)"`，漏了的话 mise 装的工具在交互 shell 里全找不到。

- **mise 装工具时也要能出网**（2026-11 实测）：
  mise 的 aqua /ubi 后端要访问 GitHub Releases，网络不通时会卡在 `verifying` 或 `downloading`。
  出网走本机代理时，把代理环境变量传给 mise：

  ```bash
  https_proxy=http://127.0.0.1:<代理端口> http_proxy=http://127.0.0.1:<代理端口> \
    mise use -g gh
  ```

- **本机状态（2026-11-21 装）**：**官方安装脚本装的**（`curl https://mise.run | sh`），
  真实路径 `~/.local/bin/mise`，版本 2026.11.10。
  更新用 `mise self-update`。
  mise 管的工具：`pnpm`、`node@22`、`gh`、`fzf`、`zoxide`、`eza`、`fd`、`ripgrep`。
  配置在 `~/.config/mise/config.toml`。

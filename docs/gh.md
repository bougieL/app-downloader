# GitHub CLI (gh)

- 命令：`gh`
- 仓库地址：https://github.com/cli/cli
- 官网：https://cli.github.com

GitHub 官方命令行工具，用来操作 PR / Issue / Release / Gist / Actions 等。

gh **没有自更新命令**（`gh upgrade` 不存在），升级只能靠当初的安装方式。

## 安装方式清单

**装的时候选了哪种，以后更新就用哪种。** 换方式的后果是同一个命令名出现两份、PATH 上互相覆盖。

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| mise | `mise ls` 里有 gh；真实路径含 `mise/installs/gh` | `mise upgrade gh` |
| GitHub Release 预编译 tar.gz | 真实路径含 `gh_<版本>_linux_<架构>/bin/gh` | 重新下载 release 里的 tar.gz，覆盖旧二进制 |
| 官方 RPM 源（cli.github.com） | `dnf repoquery --info gh` 的来源是 cli.github.com | `sudo dnf update gh` |
| 发行版自带仓库 | `dnf info gh` / `apt policy gh` 显示来源是发行版仓库 | `sudo dnf update gh`（Debian/Ubuntu：`sudo apt update && sudo apt install gh`） |
| Homebrew | 真实路径含 `/Cellar/gh` | `brew upgrade gh` |
| Conda | `conda list gh` 里有 gh | `conda update gh --channel conda-forge` |
| Nix | 真实路径含 `/nix/store/` 且包名是 gh | `nix profile upgrade gh`（或重新构建对应 flake） |

补充：

- 官方明确**不要**用 snap 装 gh（官方文档里点名 snap 运行时对 gh 问题多、建议别用）。
- Flathub 上**没有** GitHub CLI 包，所以 flatpak 这条路走不通。
- 官方 GitHub CLI apt/RPM 源的安装命令见 <https://cli.github.com/>，仓库源地址是
  `https://cli.github.com/packages`（Debian/Ubuntu 加 `/stable` 主键）。

## 检测当前安装方式

更新前必须先检测出当初是怎么装的，然后沿用它。

第一步，看真实路径（命令常常是软链或包管理器 shim，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v gh)"
```

**注意**：mise 在 PATH 上同时暴露真实路径和 shim（`~/.local/share/mise/shims/gh`），
`which -a gh` 会看到两条，**真实路径那条在前、才是生效的**。

第二步，按**路径片段**对照上表（用片段而不是完整目录，默认位置可能被环境变量或配置改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `mise/installs/gh` | mise |
| `gh_<版本>_linux_amd64/bin/gh` | GitHub Release tar.gz 手装 |
| `/Cellar/gh` | Homebrew |
| `/nix/store/` | Nix |
| `/usr/bin/gh` 或 `/usr/local/bin/gh` | 包管理器装的（看第三步确认是哪个源） |

第三步，确认包管理器来源（**路径判据不生效时以这里为准**）：

```bash
mise ls                                   # mise 装的是这里
dnf repoquery --info gh 2>/dev/null | head -20   # RPM 系看来源仓库
apt policy gh 2>/dev/null | head -20              # Debian 系看来源
brew list --versions gh 2>/dev/null
conda list gh 2>/dev/null
```

多个都命中说明装了多份，用 `which -a gh` 看 PATH 上生效的是哪个。

## 更新

**沿用检测出来的安装方式，不要换方式。**

mise 装的：

```bash
mise upgrade gh          # 升级到最新版
mise use -g gh@latest    # 固定到指定版本
```

包管理器装的：

```bash
sudo dnf update gh       # RPM 系（Fedora）
sudo apt update && sudo apt install gh   # Debian/Ubuntu 官方源
brew upgrade gh
conda update gh --channel conda-forge
```

tar.gz 手装的：重新从 release 下载对应资产（Linux 选 `gh_<版本>_linux_<架构>.tar.gz`），
用里面的 `gh_<版本>_linux_<架构>/bin/gh` 覆盖旧二进制。

## 登录

gh 默认没登录，第一次用需要认证：

```bash
gh auth login                    # 交互式，选 GitHub.com → HTTPS → 浏览器登录
gh auth login --web              # 直接走浏览器
gh auth status                   # 查看当前登录状态
```

用 `GH_TOKEN` / `GITHUB_TOKEN` 环境变量可以跳过交互登录（CI 场景常用）。

提交身份用 GitHub 的 noreply 邮箱，别用真实邮箱（真实邮箱一旦推送就永久挂在提交上）：

```bash
gh api user --jq .id             # 查自己的用户 ID
git config user.name "<用户名>"
git config user.email "<用户ID>+<用户名>@users.noreply.github.com"
```

## zsh 支持

`gh` 自带 zsh 补全：

```bash
gh completion -s zsh > ~/.local/share/zsh/site-functions/_gh
```

本机把 `~/.local/share/zsh/site-functions` 加进了 `fpath`，补全文件由
`~/.oh-my-zsh/custom/conf/project-cli.zsh` 自动生成和维护，不用手动跑。
详见 `docs/oh-my-zsh.md`。

## 注意

- mise 的 gh 走 aqua 后端（`mise registry` 里 `gh` 对应 `aqua:cli/cli`），安装时会把 release
  的 tar.gz 下载下来并校验 checksum，首次安装稍慢。
- release 的 tar.gz 有两层目录：`gh_<版本>_linux_<架构>/bin/gh`。手动装的时候别把外层目录
  整个塞进 `~/.local/bin`，只取 `bin/gh` 这一个文件。
- mise 的 shim 和真实路径会同时出现在 PATH 上，`which -a gh` 看到两条是正常的，
  前面那条（真实路径）才生效。
- gh 自己的配置在 `~/.config/gh/`（`hosts.yml` 存各 host 的 token），换机器或清理时注意别删。

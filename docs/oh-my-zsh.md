# Oh My Zsh

- 命令：`zsh`（omz 不提供独立可执行文件，靠 `~/.oh-my-zsh` 里的脚本和 `omz` 快捷命令）
- 仓库地址：https://github.com/ohmyzsh/ohmyzsh
- 官网：https://ohmyzsh.com
- 文档：https://ohmyzsh.com/docs/

zsh 的配置框架：自带插件库、主题、补全、alias 管理。**它不包含 zsh 本体**，
zsh 要由系统包管理器单独安装。

## 安装方式清单

**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方安装脚本 | `~/.oh-my-zsh` 是 git 仓库，`.git` 存在 | `git -C ~/.oh-my-zsh pull`（等价 `omz update`） |
| 手动 git clone | 同上 | `git -C ~/.oh-my-zsh pull` |
| 发行版软件包（Fedora） | `rpm -q oh-my-zsh` 有输出，`~/.oh-my-zsh` 不是 git 仓库 | `sudo dnf upgrade oh-my-zsh` |
| zinit / 其他插件管理器托管 | 仓库里没有 `.git`，由插件管理器记录版本 | 按所用插件管理器的更新命令 |

安装命令：

```bash
# 1. zsh 本体（Fedora）
sudo dnf install zsh

# 2. Oh My Zsh（官方脚本）
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

官方脚本的参数：

| 参数 | 作用 |
| --- | --- |
| `--unattended` | 不提问、不切默认 shell（`--skip-chsh` 只跳过 chsh） |
| `--skip-chsh` | 不修改默认登录 shell |
| `ZSH=<目录>` | 换安装目录，默认 `~/.oh-my-zsh` |

脚本最后会问要不要把默认 shell 换成 zsh。**它用的是 `chsh`，需要输入当前用户密码**，
自动化环境下直接跑会被卡住；免密环境用 `sudo chsh -s /usr/bin/zsh <用户名>` 效果一样：

```bash
sudo chsh -s /usr/bin/zsh "$(whoami)"
getent passwd "$(whoami)" | cut -d: -f7   # 确认已切换
```

前提是目标 shell 在 `/etc/shells` 里（Fedora 装完 zsh 包就有）。

## 检测当前安装方式

```bash
# 有没有装
ls -d ~/.oh-my-zsh 2>/dev/null

# 是不是 git 仓库（决定能不能用 git pull 更新）
test -d ~/.oh-my-zsh/.git && echo "git 安装" || echo "非 git 安装"

# 发行版包装的（此时 ~/.oh-my-zsh 通常是 /usr/share/oh-my-zsh 的软链）
rpm -q oh-my-zsh 2>/dev/null
```

三种情况互斥：git 安装 → `git pull`；发行版包装 → `dnf upgrade`；
都不是（手动复制 / 插件管理器）→ 按实际来源更新。

## 更新

**沿用检测出来的安装方式。** omz 自带 `omz update`（本质就是 `git -C ~/.oh-my-zsh pull`），
只在 git 安装时适用；发行版包装用 `dnf upgrade`，别拿 `omz update` 去动它。

```bash
omz update                     # = git -C ~/.oh-my-zsh pull
git -C ~/.oh-my-zsh pull       # 等价写法
```

## 第三方插件

omz 自带的插件在 `~/.oh-my-zsh/plugins/`，第三方插件装到 `~/.oh-my-zsh/custom/plugins/`：

```bash
git clone --depth=1 https://github.com/zsh-users/zsh-autosuggestions.git \
  ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions
```

更新第三方插件（**沿用当初的安装方式**，这里当初就是 git clone，所以还是 git）：

```bash
git -C ~/.oh-my-zsh/custom/plugins/zsh-autosuggestions pull
```

装完要在 `~/.zshrc` 的 `plugins=(...)` 里加插件名才会生效。

## 本机当前配置

- 主题：`robbyrussell`（omz 内置）。
- 第三方插件：`zsh-autosuggestions`、`zsh-syntax-highlighting`、`zsh-completions`、`fzf-tab`。
- omz 内置插件里额外启用了：`fzf`、`zoxide`、`opencode`、`codex`——前两个给 fzf / zoxide
  接上补全和快捷键，后两个**自带补全生成**（`codex completion zsh`、
  `opencode --get-yargs-completions`），不用手动配。
- 配套的 CLI 工具用 mise 装：`fzf`、`zoxide`、`eza`、`fd`、`ripgrep`。
- 自定义文件：`~/.oh-my-zsh/custom/aliases.zsh`（alias，omz 自动加载）、
  `~/.oh-my-zsh/custom/conf/project-cli.zsh`（本项目 CLI 工具的 PATH 与补全，
  在 `source $ZSH/oh-my-zsh.sh` **之前**手动 source，见下）。

## 本项目 CLI 工具的 zsh 支持

本项目装的 CLI 工具要在 zsh 下能直接用、能有补全，统一在
`~/.oh-my-zsh/custom/conf/project-cli.zsh` 里管：

1. **PATH**：把各工具的安装目录加进 `path`——
   `~/.local/bin`（脚本安装的 claude / herdr / herdr-core 等）、`~/.opencode/bin`、
   `~/.local/share/pnpm`（pnpm 全局包）、`~/.cargo/bin`，再加上 mise 的 shims。
2. **补全**：把工具生成的补全写到 `~/.local/share/zsh/site-functions/_<命令>`，
   该目录在 `fpath` 里，由 `compinit` 自动注册。

| 工具 | 补全来源 | 备注 |
| --- | --- | --- |
| `gh` | `gh completion -s zsh` | mise 装 |
| `mise` | `mise completion zsh` | mise 装 |
| `herdr` | `herdr completion zsh` | 官方脚本装 |
| `tailscale` | `tailscale completion zsh` | dnf 装 |
| `codex` | omz 内置 `codex` 插件 | 自动生成到 omz 缓存目录 |
| `opencode` | omz 内置 `opencode` 插件 | 实时问 CLI，不落盘 |
| `claude` / `pi` / `mmx` / `codebuddy` | 无 | 上游没有提供 shell 补全 |
| `herdr-core` / `hc` / `wake-cli` / `photocraft-cli` / `zed-remote-server` / `vinput` | 无 | 随桌面应用一起装的 CLI，没有补全脚本，靠 PATH 生效 |

新装一个工具时：在该文件的 `_PROJECT_CLI_COMPLETIONS` 表里加一行
`工具名 '生成补全的命令'`，补全文件会在下次开 shell 时自动生成（可执行文件比补全文件新时才重新生成）。

## 注意

- **omz 在 `source $ZSH/oh-my-zsh.sh` 里就跑完了 `compinit`，而且是在加载插件之前。**
  所以插件自己往 `fpath` 里追加的目录（比如 `zsh-completions` 的 `src/`）来不及生效，
  得自己手动加 `fpath`，否则那一堆补全函数一个都注册不上。
  本机是在 `~/.zshrc` 里显式加的：

  ```zsh
  fpath=("$HOME/.oh-my-zsh/custom/plugins/zsh-completions/src" $fpath)
  ```

  另一个办法是在插件加载完再跑一次 `compinit`，但会多花启动时间。

- **补全文件名必须以 `_` 开头。** `compinit` 只扫 fpath 里 `_#?*` 的文件，
  写成 `gh` 不会被当成补全函数，症状是「补全文件在，但按 Tab 没反应」。

- **omz 自带 `zsh-autosuggestions` 和 `zsh-syntax-highlighting`。**
  插件加载时先查 `$ZSH_CUSTOM/plugins/<名字>` 再查 `$ZSH/plugins/<名字>`，
  所以自己 clone 到 custom 目录的版本会覆盖内置版本——两套同时存在容易搞混，
  要么用内置的、要么用 custom 的，别混着来。

- **`zsh-syntax-highlighting` 必须放在 `plugins=(...)` 的最后。**
  它靠 zsh 的 `zle-line-init` 钩子工作，前面加载的插件如果也注册了钩子，
  高亮会被覆盖。同理 `zsh-autosuggestions` 要排在它前面。

- **fzf-tab 和 fzf 自带的 `fzf --zsh` 都抢 `<Tab>`。**
  fzf 的 shell 集成里有 `bindkey '^I' fzf-completion`，
  fzf-tab 加载在后面会把 `<Tab>` 改成 `fzf-tab-complete`，
  代价是 fzf 的 `**<Tab>` 模式补全用不了。`<C-r>` 历史搜索、`<C-t>` 文件搜索不受影响。

- **`zsh -i -c 'xxx'` 会刷 `can't change option: zle`。**
  这是 fzf 的 `fzf --zsh` 在没有 ZLE 会话的环境里调 `zle -N` 报的，
  真实交互终端里不会出现。脚本里跑 zsh 命令时看到这行不用当错误处理。

- **`chsh` 要密码。** 官方安装脚本在无人值守环境（`--unattended`）会跳过 chsh，
  需要手动切的话用 `sudo chsh -s /usr/bin/zsh <用户名>`。

- **mise 必须在生成补全之前激活。** `gh` 是 mise 管的，PATH 上先得有它，
  `command -v gh` 才找得到，所以 mise 的激活放在最前面。

- 卸载：omz 自带卸载脚本 `~/.oh-my-zsh/tools/uninstall.sh`；
  只想去掉某个第三方插件，删掉 `~/.oh-my-zsh/custom/plugins/<名字>` 再从 `~/.zshrc` 的
  `plugins=(...)` 里去掉对应条目。

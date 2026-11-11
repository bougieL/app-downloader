# Pi

- 命令：`pi`
- 上游仓库：https://github.com/badlogic/pi-mono（npm 包 `@mariozechner/pi-coding-agent`）
- 当前维护的 fork：https://github.com/earendil-works/pi（npm 包 `@earendil-works/pi-coding-agent`）
- 官网：https://pi.dev

终端 AI coding agent。本机常用的是 `@earendil-works/pi-coding-agent`（1.x）；上游
`@mariozechner/pi-coding-agent` 是 0.x，两条线包名不同，装之前先确认要用哪个。

## 安装方式清单

pi 没有"标准安装方式"，下面每种都有人用。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方安装脚本 | agent 目录里有 `managed-install.json` 标记 | `pi update`（脚本装的内部仍是 npm 那条路） |
| npm | 包目录路径含 `/npm/` 或 `/node_modules/` | `npm install -g --ignore-scripts --min-release-age=0 <包名>@latest` |
| pnpm | 包目录路径含 `/pnpm/` 或 `/.pnpm/` | `pnpm install -g --ignore-scripts --config.minimumReleaseAge=0 <包名>@latest` |
| bun | 包目录路径含 `/install/global/node_modules/`，或运行时是 bun | `bun install -g --ignore-scripts --minimum-release-age=0 <包名>@latest` |
| yarn | 包目录路径含 `/yarn/` 或 `/.yarn/` | `yarn global add --ignore-scripts <包名>@latest` |
| nix | `nix profile` 里注册了 pi | `nix profile upgrade pi` |

官方安装脚本（内部就是走 npm 全局安装，并写一个标记文件记录自己管着这份安装）：

```bash
curl -fsSL https://pi.dev/install.sh | sh
```

Windows：

```powershell
irm https://pi.dev/install.ps1 | iex
```

需要 Node.js 22.19 或更高。脚本装的会写标记文件到 agent 目录（默认 `~/.pi/agent/`，可用
`PI_CODING_AGENT_DIR` 环境变量改），标记文件名是 `managed-install.json`。

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看命令最终指向的**包目录**（pi 的命令是指向包内 `cli.js` 的软链，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v pi)"
```

**注意**：npm / pnpm 装出来的命令在 PATH 上可能是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
这时从 shim 里挖真实路径，或者直接查包管理器的全局列表：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v <命令>)")"
```

第二步，按**路径片段**对照上表：

| 真实路径片段 | 安装方式 |
| --- | --- |
| 含 `/pnpm/` 或 `/.pnpm/` | pnpm |
| 含 `/yarn/` 或 `/.yarn/` | yarn |
| 含 `/install/global/node_modules/` | bun |
| 含 `/npm/` 或 `/node_modules/` | npm |
| 都不匹配 | 看第三步 |

第三步，看有没有脚本安装留下的标记：

```bash
ls "<agent 目录>/install/managed-install.json"
```

默认 agent 目录是 `~/.pi/agent/`（`PI_CODING_AGENT_DIR` 可改）。有标记文件说明是官方脚本装的，
`pi update` 会按脚本那套流程来更新。

路径片段的判定规则和 pi 自己内部用的判断一致，所以按片段判断的结果和 `pi update` 的行为是对得上的。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

自带更新命令会按检测到的包管理器沿用原方式更新：

```bash
pi update                # 等价于 pi update --self，只更新 pi 自身
pi update --models       # 只刷新模型目录
pi update --all          # pi 自身 + 已装扩展
```

手动执行时注意带上 `--ignore-scripts` 和发布时间闸门参数（pi 官方发布是即时的，闸门会挡住更新）：

```bash
# npm 为例，pnpm/bun/yarn 换对应命令
npm install -g --ignore-scripts --min-release-age=0 @earendil-works/pi-coding-agent@latest
```

**pnpm 的闸门参数写法不一样**，pnpm 没有 `--min-release-age` / `--minimum-release-age` 这两个 CLI 选项
（写上去会报 `Unknown option`），要写成 `--config.minimumReleaseAge=0`：

```bash
pnpm add -g --ignore-scripts --config.minimumReleaseAge=0 @earendil-works/pi-coding-agent@latest
```

## 注意

- 两条版本线的包名不同（`@mariozechner/pi-coding-agent` vs `@earendil-works/pi-coding-agent`），
  升级到另一条线等于换包，命令名一样容易混。
- 脚本装的和 npm 手工装的可能在 PATH 上同时存在，用 `which -a pi` 查重复安装。
- 升级前退出正在运行的 pi 会话。

- node 由 mise 管着的时候，npm 的全局前缀在 mise 那个 node 的目录下（如
  `~/.local/share/mise/installs/node/<版本>/lib/node_modules`），用系统自带的 npm 升级会装到
  另一个前缀去，PATH 上还是旧版本。更新时带上 mise：

  ```bash
  mise exec node@<版本> -- npm install -g --ignore-scripts @earendil-works/pi-coding-agent@latest
  ```

- `--min-release-age=0` 这个发布时间闸门参数是 npm 11 才有的，npm 10.x 遇到它当作未知配置直接忽略，
  不会报错，也不影响升级结果（pi 官方发布是即时的，本来也不需要闸门）。

- **pnpm 全局目录里没有 `package.json`，只有一个 `pnpm-workspace.yaml`**，闸门配置写在那个文件里
  （`~/.local/share/pnpm/global/v11/pnpm-workspace.yaml`）。`pnpm config get minimumReleaseAge`
  查出来是 `undefined`，**不代表闸门没生效**——pnpm 11 对全局安装有独立的默认闸门，
  不带 `--config.minimumReleaseAge=0` 时会静默停在旧版本，装出来的版本号比 `npm view <包名> version` 报的旧。
  症状是：`pnpm add -g <包名>@latest` 显示 `+ <包名> <旧版本>`，版本号没变；
  `npm view <包名> version` 明明是新的。**每次 pnpm 更新后都要核对版本号**，
  没变就加上 `--config.minimumReleaseAge=0` 重跑。

- **`pi update` 会超时失败**（2026-11 实测）：`Error: Could not determine latest pi version:
  The operation was aborted due to timeout`。它要联网查版本，网络不通就报错而不是回退到本地。
  这时直接按上面手动执行对应包管理器的命令。

- **本机状态（2026-11-10 核实）**：pnpm 装的（`~/.local/share/pnpm/global/v11`），
  实测从 1.5.0 升到 1.5.2 就是靠 `--config.minimumReleaseAge=0` 才升上去的，
  不带这个参数时 `pnpm add -g ...@latest` 装完还是 1.5.0。

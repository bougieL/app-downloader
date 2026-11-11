# MMX CLI

- 命令：`mmx`
- 仓库地址：https://github.com/MiniMax-AI/cli
- npm 包：`mmx-cli`
- 官网：https://platform.minimax.io

MiniMax 官方 CLI，用来在终端调用 MiniMax 的文本、图片、视频、语音、音乐、搜索等能力，
也可以把 MiniMax 配置写进 Claude Code / Codex / OpenCode / Pi 等 agent。

要求 Node.js >= 18，并且需要 MiniMax Token Plan（[Global](https://platform.minimax.io/subscribe/token-plan) /
[CN](https://platform.minimaxi.com/subscribe/token-plan)）。

## 安装方式清单

CLI 本体只有一个 npm 包 `mmx-cli`，装在哪个包管理器里就用什么管理器更新。
**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| npm | 全局列表里有 `mmx-cli`，真实路径含 `/npm/` 或 `/node_modules/` | `npm update -g mmx-cli` |
| pnpm | 全局列表里有 `mmx-cli`，真实路径含 `/pnpm/` | `pnpm update -g mmx-cli` |
| bun | 全局列表里有 `mmx-cli`，真实路径含 `/bun/` 或 `/install/global/node_modules/` | `bun update -g mmx-cli`（旧版 bun 用 `bun install -g mmx-cli@latest`） |
| yarn | 全局列表里有 `mmx-cli`，真实路径含 `/yarn/` 或 `/.yarn/` | `yarn global add mmx-cli@latest` |

安装命令：

```bash
npm install -g mmx-cli    # npm
pnpm add -g mmx-cli       # pnpm
bun add -g mmx-cli        # bun
yarn global add mmx-cli   # yarn
```

另外还有一种**只装 agent skill、不装 CLI** 的方式（`skills` CLI）：

```bash
npx skills add MiniMax-AI/cli -y -g
```

它把 `SKILL.md` 之类的文件放进各 agent 的 skills 目录，跟 `mmx` 命令本身是两回事。
用这个方式装过 skill 的，更新 skill 走 `npx skills update MiniMax-AI/cli`（或重新 `npx skills add MiniMax-AI/cli -y -g`），
不要跟 CLI 的更新混在一起。

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（命令常常是 shim 脚本，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v mmx)"
```

**注意**：npm / pnpm 装出来的命令在 PATH 上是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
这时从 shim 里挖真实路径，或者直接查包管理器的全局列表：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v mmx)")"
```

pnpm v10+ 的全局目录带哈希子目录（形如 `.../pnpm/global/v<版本>/<hash>/node_modules/mmx-cli/`），
路径片段里仍然含 `/pnpm/`，按片段判断即可。

第二步，按**路径片段**对照上表：

| 真实路径片段 | 安装方式 |
| --- | --- |
| 含 `/pnpm/` | pnpm |
| 含 `/yarn/` 或 `/.yarn/` | yarn |
| 含 `/bun/` 或 `/install/global/node_modules/` | bun |
| 含 `/npm/` 或 `/node_modules/` | npm 系，继续第三步 |
| 都不匹配 | 看第三步 |

第三步，区分是哪个包管理器（**这一步最可靠，路径判据对 shim 不生效时以这里为准**）：

```bash
npm ls -g --depth=0 2>/dev/null | grep "mmx-cli"
pnpm ls -g --depth=0 2>/dev/null | grep "mmx-cli"
bun pm ls -g 2>/dev/null | grep "mmx-cli"
yarn global list 2>/dev/null | grep "mmx-cli"
```

哪个列出了 `mmx-cli`，就是哪个包管理器装的。多个都列出来说明机器上装了多份，
用 `which -a mmx` 看 PATH 上到底生效的是哪个。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

自带的 `mmx update` **不做包管理器检测**，只会打印一句 `npm update -g mmx-cli`
（1.0.16 和 1.0.28 实测都一样，带 `latest` 参数也一样）。它是提示不是动作——当初用 pnpm 装的照着它做，
会把包装到 npm 的全局前缀下，PATH 上出现两个 `mmx`。所以**忽略它打印的提示，按检测结果手动执行**：

```bash
# npm 为例，其余换对应命令
npm update -g mmx-cli

# pnpm
pnpm add -g --ignore-scripts --config.minimumReleaseAge=0 mmx-cli@latest

# bun
bun update -g mmx-cli

# yarn
yarn global add mmx-cli@latest
```

**pnpm 的闸门参数只能写成 `--config.minimumReleaseAge=0`**（2026-11-10 实测）：
`--min-release-age=0` / `--minimum-release-age=0` 都会被 pnpm 11 拒绝（报 `Unknown option`）。
**不带闸门参数时 `pnpm add -g mmx-cli@latest` 会静默停在旧版本**：
本机当时 npm 上最新是 1.0.29，不带参数只装到 1.0.28，加了 `--config.minimumReleaseAge=0`
才升到 1.0.29。`pnpm config get minimumReleaseAge` 查出来是 `undefined`，**不代表闸门没生效**。

更新完确认一下版本和生效的路径：

```bash
mmx --version
which -a mmx
```

## 版本差异

CLI 版本跨度比较大，子命令有增补。下面这些是新版才有的，脚本里要用到时先确认版本：

| 版本 | 新增内容 |
| --- | --- |
| 1.0.28 | `mmx speech transcribe`（语音转写）、`mmx agent setup`（把 MiniMax 配置写进各 agent）、`mmx file`（文件上传/列表/删除）、`mmx search` 简写 |
| 1.0.28 | 文本默认模型换成 `MiniMax-M3.1-Flash-Preview`（需要 M Plan 或 MiniMax Code；按量付费的 key 要显式 `--model MiniMax-M3`） |

1.0.28 还新增了 `mmx agent setup`，交互式向导里能顺手安装没装在 PATH 上的 agent，
并会把原配置备份后再改写——脚本里跑记得加 `--dry-run` 先看要写什么。

## 配置与凭据

```bash
mmx auth login                      # 交互式，OAuth 凭据存 ~/.mmx/credentials.json
mmx auth login --api-key sk-xxxxx   # API key 存 ~/.mmx/config.json
mmx auth status                     # 看当前用的是哪种凭据

mmx config show
mmx config set --key region --value cn
```

环境变量优先级高于配置文件：`MINIMAX_API_KEY`、`MINIMAX_REGION`（`global` / `cn`，
不设置时自动探测）。把 API key 写进环境变量或本地配置文件，不要提交到仓库。

## 注意

- `mmx update` 硬编码 npm 命令，不管当初是怎么装的，别直接照抄。实测 1.0.16 和 1.0.28 都是这样，
  `mmx update latest` 也不会真的去更新。
- node 由 mise 管着的时候，npm 的全局前缀在 mise 那个 node 的目录下（如
  `~/.local/share/mise/installs/node/<版本>/lib/node_modules`），用系统自带的 npm 升级会装到
  另一个前缀去，PATH 上还是旧版本。更新时带上 mise：

  ```bash
  mise exec node@<版本> -- npm update -g mmx-cli
  ```

- pnpm / bun / yarn 装的不受 mise node 前缀影响，只要 `node` 在 PATH 上就能跑。
- 需要 Node.js >= 18；版本太低会在启动时报 engines 错误。
- 同一台机器装过多个 `mmx` 时用 `which -a mmx` 查重复安装。
- **本机状态（2026-11-10 核实）**：pnpm 装的（`~/.local/share/pnpm/global/v11`），
  从 1.0.28 升到 1.0.29，同样是靠 `--config.minimumReleaseAge=0` 才升上去的。

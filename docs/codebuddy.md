# CodeBuddy Code

- 命令：`codebuddy`（别名 `cbc`、`codebuddy-code`）
- npm 包：`@tencent-ai/codebuddy-code`
- 仓库地址：https://cnb.cool/codebuddy/codebuddy-code
- 官网：https://copilot.tencent.com/cli
- 文档：https://www.codebuddy.ai/docs/cli

腾讯官方的终端编码助手，能读代码、改文件、跑命令，也提供 IDE 插件、GitHub 集成和 Agent SDK。

## 安装方式清单

CLI 有两套分发：**Node.js 包**（npm / pnpm / yarn / bun / Homebrew）和**原生二进制**（Beta）。
装的时候选了哪种，以后更新就用哪种。

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| npm | 全局列表里有 `@tencent-ai/codebuddy-code`，真实路径含 `/npm/` 或 `/node_modules/` | `npm update -g @tencent-ai/codebuddy-code` |
| pnpm | 全局列表里有 `@tencent-ai/codebuddy-code`，真实路径含 `/pnpm/` | `pnpm update -g @tencent-ai/codebuddy-code` |
| yarn | 全局列表里有 `@tencent-ai/codebuddy-code`，真实路径含 `/yarn/` 或 `/.yarn/` | `yarn global add @tencent-ai/codebuddy-code@latest` |
| bun | 全局列表里有 `@tencent-ai/codebuddy-code`，真实路径含 `/bun/` 或 `/install/global/node_modules/` | `bun update -g @tencent-ai/codebuddy-code` |
| Homebrew | `brew list --formula` 里有 `codebuddy-code`，真实路径含 `/Cellar/` 或 `/homebrew/` | `brew upgrade codebuddy-code` |
| 原生二进制 | 真实路径是 ELF / Mach-O 可执行文件，不是脚本，路径里没有 `node_modules` | `codebuddy update` |

安装命令：

```bash
npm install -g @tencent-ai/codebuddy-code    # npm
pnpm add -g @tencent-ai/codebuddy-code        # pnpm
yarn global add @tencent-ai/codebuddy-code    # yarn
bun add -g @tencent-ai/codebuddy-code         # bun

brew tap Tencent-CodeBuddy/tap               # Homebrew（macOS/Linux，无需 Node.js）
brew install codebuddy-code
```

原生二进制（Beta，无需 Node.js，单文件）：

```bash
curl -fsSL https://copilot.tencent.com/cli/install.sh | bash   # macOS / Linux
# Windows: irm https://copilot.tencent.com/cli/install.ps1 | iex
```

已经用包管理器装好的，可以用 `codebuddy install` 迁移到原生二进制版。

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（命令常常是 shim 脚本，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v codebuddy)"
file "$(readlink -f "$(command -v codebuddy)")"
```

- `file` 报 `ELF` / `Mach-O` → **原生二进制**
- `file` 报 `POSIX shell script` / `sh` → **包管理器装的**，继续第二步挖真实路径：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v codebuddy)")"
```

第二步，按**路径片段**对照上表：

| 真实路径片段 | 安装方式 |
| --- | --- |
| 含 `/pnpm/` | pnpm |
| 含 `/yarn/` 或 `/.yarn/` | yarn |
| 含 `/bun/` 或 `/install/global/node_modules/` | bun |
| 含 `/Cellar/` 或 `/homebrew/` | Homebrew |
| 含 `/npm/` 或 `/node_modules/` | npm 系，继续第三步 |
| 都不匹配 | 看第三步 |

第三步，区分是哪个包管理器（**这一步最可靠，路径判据对 shim 不生效时以这里为准**）：

```bash
npm ls -g --depth=0 2>/dev/null | grep "codebuddy-code"
pnpm ls -g --depth=0 2>/dev/null | grep "codebuddy-code"
bun pm ls -g 2>/dev/null | grep "codebuddy-code"
yarn global list 2>/dev/null | grep "codebuddy-code"
brew list --formula 2>/dev/null | grep "codebuddy-code"
```

哪个列出了 `@tencent-ai/codebuddy-code`，就是哪个包管理器装的。多个都列出来说明机器上装了多份，
用 `which -a codebuddy` 看 PATH 上到底生效的是哪个。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

原生二进制版用自带命令更新（它就是原生安装器，自带自动更新）：

```bash
codebuddy update
```

包管理器版**按检测结果手动执行对应那一条**。`codebuddy update` 会自行检测安装方式，
但检测结果不一定准（尤其是同机装过多种包管理器时），要跟上面的检测结果核对；
不一致就以检测结果为准：

```bash
# npm
npm update -g @tencent-ai/codebuddy-code

# pnpm
pnpm update -g @tencent-ai/codebuddy-code

# yarn
yarn global add @tencent-ai/codebuddy-code@latest

# bun
bun update -g @tencent-ai/codebuddy-code

# Homebrew
brew upgrade codebuddy-code
```

更新完确认一下版本和生效的路径：

```bash
codebuddy --version
which -a codebuddy
```

## 登录

首次启动会弹登录方式选择：国内站（copilot.tencent.com）、国际站（codebuddy.ai）、
企业专享版（自建服务，需要填服务地址）、腾讯内部 iOA。选完自动开浏览器完成认证。

## 配置目录

| 平台 | 默认目录 |
| --- | --- |
| macOS / Linux | `~/.codebuddy` |
| Windows | `%USERPROFILE%\.codebuddy` |

里面放 `settings.json`、`.mcp.json`、`skills/` 等。可以用 `CODEBUDDY_CONFIG_DIR` 改位置：

```bash
export CODEBUDDY_CONFIG_DIR="$HOME/.my-codebuddy-config"
```

## 注意

- **原生二进制版和包管理器版都往 `~/.local/bin/codebuddy` 放文件**（原生版是直接放可执行文件，
  包管理器版是 pnpm / npm 生成的 shim 脚本）。想换方式的话先把旧的卸掉，
  否则同一个名字两份、PATH 上互相覆盖。原生版卸载就是删掉那个可执行文件。
- pnpm v11 的全局目录带哈希子目录（形如 `.../pnpm/global/v11/<hash>/node_modules/@tencent-ai/codebuddy-code/`），
  路径片段里仍然含 `/pnpm/`，按片段判断即可。
- node 由 mise 管着的时候，npm 的全局前缀在 mise 那个 node 的目录下（如
  `~/.local/share/mise/installs/node/<版本>/lib/node_modules`），用系统自带的 npm 升级会装到
  另一个前缀去，PATH 上还是旧版本。更新时带上 mise：

  ```bash
  mise exec node@<版本> -- npm update -g @tencent-ai/codebuddy-code
  ```

  pnpm / bun / yarn 装的不受 mise node 前缀影响，只要 `node` 在 PATH 上就能跑。
- 默认开启自动更新（原生版靠它更新自己）。要关掉设 `DISABLE_AUTOUPDATER=1`。
- `codebuddy doctor` 是交互式检查，**在脚本或非交互终端里跑会挂住不返回**，别放进自动化流程。
- 需要 Node.js >= 18.20（原生二进制版不需要 Node.js）。
- `codebuddy -p "..."` 走单次输出；要让它动文件、跑命令，得额外加 `-y`
  （`--dangerously-skip-permissions`）。

- **pnpm 全局装的版本可能被精确锁死，`pnpm update -g` 会「Already up to date」不动**：
  用 `pnpm add -g <包>@<版本>` 装的话，全局 manifest 里写的是精确版本
  （如 `"@tencent-ai/codebuddy-code": "2.158.0"`），`pnpm update -g <包>` 认为没超范围就不动。
  查当前写法：

  ```bash
  grep -rl --include=package.json "codebuddy-code" ~/.local/share/pnpm/global/v*/ | head
  ```

  精确锁死的要用 `pnpm add -g <包>@<新版本>` 更新；带 `^` 范围的才能用 `pnpm update -g`。
- **pnpm 的发布时间闸门会挡住刚发布的版本**：pnpm 12 按发布时间过滤新包，
  `pnpm outdated -g` 给出的 Latest 可能比 `npm view <包> version` 旧（例：npm 上是 2.181.0，
  pnpm 认为最新是 2.180.0）。想装被挡住的版本，要么等闸门过去，要么把它加进全局
  `~/.local/share/pnpm/global/v*/pnpm-workspace.yaml` 的 `minimumReleaseAgeExclude` 列表。
- **精确锁死后只能用 `pnpm add -g <包>@<版本>` 更新，`pnpm update -g` 不动**：本机全局 manifest
  里写的是 `"@tencent-ai/codebuddy-code": "2.180.0"`（无 `^`），实测 `pnpm outdated -g` 能列出
  2.181.0 但 `pnpm update -g` 会判定没超范围；改用 `pnpm add -g @tencent-ai/codebuddy-code@2.181.0`
  成功升级。升级前先 `grep -rh "codebuddy-code" ~/.local/share/pnpm/global/v*/package.json`
  看是精确版本还是 `^` 范围。
- **镜像源下载 error (23) 会自动重试成功**：走 npmmirror 镜像时 `pnpm add -g` 可能报
  `[WARN] GET https://registry.npmmirror.com/@tencent-ai/codebuddy-code/-/codebuddy-code-<版本>.tgz error (23). Will retry in 10 seconds. 2 retries left.`，
  pnpm 自动重试后仍会装成功，不用中断重跑；看到这行等它跑完核对 `codebuddy --version` 即可。

- **pnpm 闸门参数只能写成 `--config.minimumReleaseAge=0`**（2026-11-10 实测）：
  `--min-release-age=0` 和 `--minimum-release-age=0` 都会被 pnpm 11 拒绝，
  报 `Unknown option: 'min-release-age'`。pnpm 11 没有这两个 CLI 选项，
  要用 `--config.` 前缀传配置项：

  ```bash
  pnpm add -g --ignore-scripts --config.minimumReleaseAge=0 @tencent-ai/codebuddy-code@latest
  ```

  **不带这个参数时 `pnpm add -g <包>@latest` 会静默停在旧版本**：
  2026-11-10 实测 npm 上最新是 2.182.0，不带参数只装到 2.181.0，
  加了 `--config.minimumReleaseAge=0` 才升到 2.182.0。`pnpm config get minimumReleaseAge`
  查出来是 `undefined`，**不代表闸门没生效**，别据此判断。

- **本机状态（2026-11-10 核实）**：pnpm 装的（`~/.local/share/pnpm/global/v11`），
  从 2.180.0 → 2.181.0 → 2.182.0。中间 2.181.0 那次是 `pnpm add -g --ignore-scripts`（不带闸门参数）
  装的，2.182.0 是加 `--config.minimumReleaseAge=0` 装的。

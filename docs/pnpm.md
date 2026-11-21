# pnpm

- 命令：`pnpm`
- 仓库地址：https://github.com/pnpm/pnpm
- 官网：https://pnpm.io

Node.js 生态的包管理器，也是本项目**首选的包管理器**（AGENTS.md：包管理优先 pnpm 而非 npm）。

## 安装方式清单

pnpm 有很多种装法。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| mise（`mise use -g pnpm`） | 真实路径在 `~/.local/share/mise/installs/pnpm/` 下，PATH 上是 `~/.local/share/mise/shims/pnpm` | `mise upgrade pnpm` 或 `mise use -g pnpm@latest` |
| 官方 standalone 脚本 | 真实路径是 `~/.local/share/pnpm/pnpm`（脚本默认装这里） | 重新跑安装脚本，或下载新版本 tarball 覆盖 |
| corepack（随 Node.js 附带） | `corepack enable pnpm` 后 PATH 上是 node 安装目录里的 `pnpm` shim | `corepack use pnpm@latest`（改 `package.json` 的 `packageManager` 字段） |
| npm 全局 | `npm ls -g --depth=0` 里有 `pnpm`，PATH 上是 npm 全局目录里的 shim | `npm install -g pnpm@latest` |
| pnpm 全局（自举） | `pnpm ls -g --depth=0` 里有 `pnpm` | `pnpm add -g pnpm@latest` |
| Homebrew | 真实路径含 `/Cellar/pnpm` | `brew upgrade pnpm` |
| Windows 其它方式（scoop / winget / choco） | `scoop list pnpm` / `winget list pnpm` 有输出 | 用装它的那个管理器更新 |

注意：

- pnpm 官方 Release 里有多种资产，跨平台命名不同；mise 装的是 **standalone 压缩包**，
  不带 Node.js 运行时（自带一个精简的 node 运行时），所以不需要先装 Node。
- corepack 那条路只在项目里 `package.json` 写了 `packageManager` 字段才生效，
  且 Node.js 版本太老（< 16.9）没有 corepack。

## 检测当前安装方式

```bash
command -v pnpm
readlink -f "$(command -v pnpm)"
```

按**路径片段**对照上表（用片段而不是完整目录，默认位置可以被环境变量或配置改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `/mise/installs/pnpm/` | mise |
| `/mise/shims/pnpm` | mise（shim，指向 `~/.local/share/mise/installs/pnpm/`） |
| `/.local/share/pnpm/pnpm` | 官方 standalone 脚本 |
| `/node_modules/pnpm/` | npm / pnpm 全局安装 |
| `/Cellar/pnpm` | Homebrew |

mise 装的时候 PATH 上是 shim 脚本，`readlink -f` 会停在 shim 自己，
这种情况用 `mise where pnpm` 拿真实路径：

```bash
mise where pnpm        # 例如 ~/.local/share/mise/installs/pnpm/12.21.0
mise ls pnpm           # 已安装版本
```

npm / pnpm 装出来的命令在 PATH 上同样是 shim 脚本，路径判断不了的时候以全局列表为准：

```bash
npm ls -g --depth=0
pnpm ls -g --depth=0
```

判断不出来就问用户，不要猜，更不要擅自换方式重装。

## 更新

**沿用检测出来的那种方式更新。**

mise 装的：

```bash
mise upgrade pnpm            # 升级到最新版并更新 config.toml 里的记录
mise use -g pnpm@<版本>       # 指定版本
mise ls-remote pnpm          # 看远端可安装的版本
```

standalone 脚本 / npm 全局 / corepack 装的，分别按上面「安装方式清单」里那一行执行。

## 常用命令

```bash
pnpm -v                      # 版本
pnpm init                    # 新建 package.json
pnpm add <包名>               # 加依赖
pnpm add -g <包名>            # 装全局包
pnpm install                 # 按 lockfile 安装
pnpm dlx <包名>               # 临时执行一次性脚本
pnpm store path              # 全局 store 路径
```

## 全局包目录

pnpm 的全局包在 `~/.local/share/pnpm/global/`（`pnpm root -g`），
**全局命令（可执行文件）在 `~/.local/share/pnpm/bin/`**（`pnpm bin -g`）——
这两个目录不是同一个，`bin/` 里才是 `claude`、`codex`、`mmx`、`codebuddy` 这些命令。

`~/.local/share/pnpm/bin` 已经在 `~/.oh-my-zsh/custom/conf/project-cli.zsh` 的 `path` 里，
所以全局装的命令在 zsh 下可以直接用。

不装 pnpm 也能用，但 pnpm 会直接报：

```
Error: ERR_PNPM_GLOBAL_BIN_DIR_NOT_IN_PATH
  × The configured global bin directory ".../pnpm/bin" is not in PATH
```

要么把那个目录加进 PATH，要么跑一次 `pnpm setup`（它会改 shell 配置）。

## zsh 支持

pnpm 自带 zsh 补全：

```bash
pnpm completion zsh
```

输出的第一行是 `#compdef pnpm pn`（一次声明了两个补全目标），
由 `~/.oh-my-zsh/custom/conf/project-cli.zsh` 自动生成到
`~/.local/share/zsh/site-functions/_pnpm`，不用手动跑。
详见 `docs/oh-my-zsh.md`。

## 注意

- mise 里 pnpm **没有** `install -g` 这个写法：`mise install` 只接受工具名，
  想装全局的用 `mise use -g <工具>@<版本>`（`mise use -g` 会同时写进
  `~/.config/mise/config.toml`）。写错会报 `unexpected argument '-g' found`。
- mise 的 `use -g` 记的是 `latest`，`mise upgrade` 之后 config.toml 里仍是 `latest`，
  实际版本用 `mise ls pnpm` 或 `mise where pnpm` 看。
- mise 装 pnpm 时不需要 Node.js；但 `pnpm add` 装的项目依赖会带上各自的 node 依赖，
  要跑项目脚本时项目里通常还是得自己 `mise use --node <版本>`。
- pnpm 的全局 store 默认在 `~/.local/share/pnpm/store`，
  装包多时占空间，`pnpm store prune` 可以清理没被引用的包。

- **pnpm 12 默认不跑依赖的 build 脚本**（2026-11 实测）：
  `claude`、`codex` 这类包的原生二进制靠 postinstall 就位，
  不显式允许就装出一个「命令在、跑起来报错」的状态：

  ```
  Error: claude native binary not installed.
  Either postinstall did not run (--ignore-scripts, some pnpm configs) ...
  ```

  装这类包要加 `--allow-build=<包名>`：

  ```bash
  pnpm add -g --config.minimumReleaseAge=0 --allow-build=@anthropic-ai/claude-code \
    @anthropic-ai/claude-code@latest
  ```

  **别用 `--ignore-scripts`**：加了 postinstall 一定不跑，必挂。

- **pnpm 12 的发布时间闸门参数只能写成 `--config.minimumReleaseAge=0`**：
  `--min-release-age=0` / `--minimum-release-age=0` 都会被拒绝（报 `Unknown option`）。
  不带这个参数时 `pnpm add -g <包>@latest` 会**静默停在旧版本**，
  装完记得核对版本号。

- **本机状态（2026-11-21 装）**：**mise 装的** `pnpm@12.21.0`，
  PATH 上是 `~/.local/share/mise/shims/pnpm`，
  全局包目录 `~/.local/share/pnpm/global/v11`，全局命令目录 `~/.local/share/pnpm/bin`。
  registry 指向 `https://registry.npmmirror.com/`（见 `docs/mirrors.md`）。

- **npm 全局列表里的 `npm` 和 `corepack` 是 Node.js 自带的，不是用户装的包**
  （就在 node 安装目录的 `lib/node_modules` 里，和 node 本体一起发布）。
  `npm ls -g --depth=0` 会把它们列出来，别误当成「还有 npm 装的全局包要迁到 pnpm」——
  它们迁不了也不该迁，node 本体不归 pnpm 管。
  本机（2026-11-21）已把用户装的全局包全部迁到 pnpm，
  npm 全局只剩 `npm` 和 `corepack` 这两个自带的。

- **pnpm 12 的全局目录是哈希布局**：`~/.local/share/pnpm/global/v11/` 下不是
  `node_modules/<包名>`，而是一堆 `<hash>-<hash>-0` 目录 + 一个 `pnpm-workspace.yaml`，
  包名对不上目录名。查全局包用 `pnpm ls -g --depth=0`，别去猜目录名。

# OpenCode

- 命令：`opencode`
- 仓库地址：https://github.com/anomalyco/opencode
- 官网：https://opencode.ai

开源终端 AI coding agent。仓库原为 `sst/opencode`，现已迁到 `anomalyco/opencode`。

## 包名与版本线（容易踩坑）

| 版本线 | npm 包名 | 说明 |
| --- | --- | --- |
| OpenCode 1 | `opencode-ai` | 只发 1.x，最新 1.19.x |
| OpenCode 2 | `@opencode/cli` | 发 2.x，命令名仍是 `opencode` |

两者装出来的可执行文件都叫 `opencode`，在 PATH 上互相覆盖。跨版本线升级会换包名，
下面所有检测和更新都要先确认当前是哪条线。

## 安装方式清单

opencode 没有"标准安装方式"，下面每种都有人用。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 安装脚本 | 真实路径含 `/.opencode/bin/` | `curl -fsSL https://opencode.ai/v2/install \| bash` |
| npm | 全局列表里有 `opencode-ai` 或 `@opencode/cli` | `npm install -g <包名>@latest`（1.x 是 `opencode-ai`，2.x 是 `@opencode/cli`） |
| pnpm | 全局列表里有对应包 | `pnpm add -g <包名>@latest`；2.x 需要 `pnpm add -g --allow-build=@opencode/cli @opencode/cli@latest` |
| bun | 全局列表里有对应包 | `bun install -g --trust <包名>@latest` |
| brew | 真实路径含 `/Cellar/opencode` | `brew upgrade <当初装的那个 formula>` |
| mise | `mise ls` 里有 opencode | `mise upgrade opencode` |

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（命令常常是软链或包管理器 shim，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v opencode)"
```

**注意**：npm / pnpm 装出来的命令在 PATH 上是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
这时要从 shim 里挖出真实路径，或者直接查包管理器的全局列表：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v opencode)")"
```

第二步，按**路径片段**对照上表（用片段而不是完整目录，因为默认位置可以被环境变量或配置改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `/.opencode/bin/` | 安装脚本（默认装到 `$HOME/.opencode/bin/`） |
| `/Cellar/opencode` | brew |
| `node_modules/opencode-ai/` | npm 系（1.x），继续第三步 |
| `node_modules/@opencode/cli/` | npm 系（2.x），继续第三步 |
| 落在 `~/.local/bin/` 且不匹配上面任何一项 | 1.x 自身逻辑会把 `.local/bin` 判定成安装脚本，需要再确认 |

第三步，区分是哪个包管理器（**这一步最可靠，路径判据对 shim 不生效时以这里为准**）：

```bash
npm ls -g --depth=0 2>/dev/null | grep -E "opencode-ai|@opencode/cli"
pnpm ls -g --depth=0 2>/dev/null | grep -E "opencode-ai|@opencode/cli"
bun pm ls -g 2>/dev/null | grep -E "opencode-ai|@opencode/cli"
```

哪个列出了包名，就是哪个包管理器装的。多个都列出来，说明机器上装了多份，用 `which -a opencode`
看 PATH 上到底生效的是哪个。

`opencode upgrade` 内部也做同样的检测，会打印 `Using method: <method>`，可以拿它交叉验证。
它支持的 method 取值以 `opencode upgrade --help` 的 `choices` 为准（1.x 是 `curl/npm/pnpm/bun/brew/choco/scoop`）。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

自带的更新命令会沿用原方式更新，但它默认自动检测，检测不出来时需要手动指定：

```bash
opencode upgrade                          # 自动检测
opencode upgrade --method pnpm            # 检测结果和实际不符时，按检测结果显式指定
opencode upgrade 1.19.5                   # 指定目标版本
```

自带的更新命令检测不出来（报 "Could not detect the installation method"）或者和第 2、3 步的检测结果
不一致时，按上表的「沿用该方式的更新命令」手动执行对应那一条。

从 1.x 升到 2.x 会换包名（`opencode-ai` → `@opencode/cli`），包管理器不变：

```bash
# pnpm 为例，顺序不能反：先卸 1.x 再装 2.x
pnpm remove -g opencode-ai
pnpm add -g --allow-build=@opencode/cli @opencode/cli
```

`opencode upgrade --major` 做的是同一件事（先卸载 1.x，装 2.x 失败会回滚）。

## 注意

- 同一台机器上装过 1.x 和 2.x 会出现两个 `opencode`，用 `which -a opencode` 查重复安装。
- 升级前退出正在运行的 opencode 会话。

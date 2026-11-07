# Codex CLI

- 命令：`codex`
- 仓库地址：https://github.com/openai/codex
- 官方文档：https://developers.openai.com/codex

OpenAI 的终端 coding agent（Rust 写的，npm 包里的 `codex.js` 只是 shim，真正跑的是原生二进制）。

## 安装方式清单

codex 没有"标准安装方式"，下面每种都有人用。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| standalone 安装脚本 | 真实路径含 `packages/standalone/` | `sh -c 's=$(curl -fsSL https://chatgpt.com/codex/install.sh) && CODEX_NON_INTERACTIVE=1 eval "$s"'` |
| npm | 全局列表里有 `@openai/codex` | `npm install -g @openai/codex@latest` |
| pnpm | 全局列表里有 `@openai/codex` | `pnpm add -g @openai/codex@latest` |
| bun | 全局列表里有 `@openai/codex` | `bun install -g @openai/codex@latest` |
| brew | macOS 上真实路径在 `/opt/homebrew` 或 `/usr/local` 下 | `brew upgrade --cask codex` |
| 手动放二进制 | 真实路径不匹配上面任何一项 | 自己替换二进制文件 |

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（命令通常是软链，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v codex)"
```

**注意**：npm / pnpm 装出来的命令在 PATH 上可能是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
这时从 shim 里挖真实路径，或者直接查包管理器的全局列表：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v <命令>)")"
```

第二步，按**路径片段**对照上表（用片段而不是完整目录，因为 codex 的家目录可以被 `CODEX_HOME` 改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `packages/standalone/` | standalone 安装脚本（家目录默认 `~/.codex/`，设了 `CODEX_HOME` 就是那个值） |
| `node_modules/@openai/codex/` | npm 系，继续第三步 |
| macOS 且在 `/opt/homebrew` 或 `/usr/local` 下 | brew |
| 其它 | 手动放二进制 |

第三步，区分是哪个包管理器：

```bash
npm ls -g --depth=0 2>/dev/null | grep "@openai/codex"
pnpm ls -g --depth=0 2>/dev/null | grep "@openai/codex"
bun pm ls -g 2>/dev/null | grep "@openai/codex"
```

哪个列出了包名，就是哪个包管理器装的。多个都列出来说明装了多份，用 `which -a codex` 看生效的是哪个。

codex 自己的判定和上面一致：npm/pnpm/bun 装的 shim 会带 `CODEX_MANAGED_BY_NPM` / `CODEX_MANAGED_BY_PNPM` /
`CODEX_MANAGED_BY_BUN` 环境变量，standalone 靠路径里的 `packages/standalone` 识别。

standalone 装的话可以再确认一下版本软链：

```bash
ls -l "<codex 家目录>/packages/standalone/current"
```

`current` 是指向 `releases/<版本>-<目标平台>` 的软链。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

自带更新命令按检测结果沿用原方式更新：

```bash
codex update
```

它检测不出来时直接报错：

```
Could not detect the Codex installation method. Please update manually: https://developers.openai.com/codex/cli/
```

这时按上表的「沿用该方式的更新命令」手动执行对应那一条。

## 注意

- standalone 安装按版本分目录存：`<codex 家目录>/packages/standalone/releases/<版本>-<目标平台>/`，
  `current` 软链指向当前版本。升级会新开一个版本目录并切换软链，旧版本目录会留下来，需要时自己清理。
- npm/pnpm/bun 装的 `@openai/codex` 里 `codex.js` 只是 shim，真正的二进制在同包的 `vendor/` 下。
- 升级前退出正在运行的 codex 会话。

- **standalone 更新的安装脚本要从 `https://chatgpt.com/codex/install.sh` 拉取，直连不通时要显式走代理**：

  ```bash
  HTTPS_PROXY=http://127.0.0.1:<代理端口> HTTP_PROXY=http://127.0.0.1:<代理端口> codex update
  ```

  没有代理环境变量时，`codex update` 会在 curl 上卡到超时并报
  `curl: (28) Failed to connect to chatgpt.com port 443`，版本不变。
  走代理后整个流程要几分钟（脚本重下 + 拉几十 MB 的 release 资产），
  中途别打断，否则会留下 `~/.codex/packages/standalone/install.lock` 和半截的版本目录。
  脚本本身下载的资产在 `openai/codex` 的 `rust-v<版本>` release 上（如 `rust-v0.167.0`），
  资产名 `codex-package-<目标平台>.tar.gz`。

- **下载超时会自动回退到 GitHub Releases，脚本仍以成功结束**：走 `releases.openai.com` 拉 200MB
  的 `codex-package-<目标平台>.tar.gz` 时可能 300 秒超时（报
  `curl: (28) Operation timed out after 300000 milliseconds with ... out of ... bytes received`），
  脚本会打印 `WARNING: Could not download or verify ...; retrying from GitHub Releases.`
  再从 `github.com/openai/codex/releases` 重下一次装好。看到这行不用当失败处理，
  等它跑完确认 `codex --version` 变了即可。

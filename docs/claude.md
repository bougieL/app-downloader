# Claude Code

- 命令：`claude`
- 仓库地址：https://github.com/anthropics/claude-code
- 官方文档：https://code.claude.com/docs

Anthropic 的终端 coding agent。npm 包 `@anthropic-ai/claude-code` 里的 `claude` 也只是 shim，
真正跑的是原生二进制。

## 安装方式清单

claude code 没有"标准安装方式"，下面每种都有人用。**装的时候选了哪种，以后更新就用哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 原生安装脚本 | 真实路径含 `claude/versions/` | `claude update` |
| npm | 全局列表里有 `@anthropic-ai/claude-code` | `npm install -g @anthropic-ai/claude-code@latest` |
| pnpm | 全局列表里有 `@anthropic-ai/claude-code` | `pnpm add -g @anthropic-ai/claude-code@latest` |
| bun | 全局列表里有 `@anthropic-ai/claude-code` | `bun install -g @anthropic-ai/claude-code@latest` |
| brew | `brew list --cask` 里有 `claude-code` 或 `claude-code@latest` | `brew upgrade claude-code` / `brew upgrade claude-code@latest` |
| winget（Windows） | `winget list` 里有 `Anthropic.ClaudeCode` | `winget upgrade Anthropic.ClaudeCode` |
| apt/dnf/apk | 系统包管理器能列出 `claude-code` 的文件 | 用系统包管理器升级（需要 sudo） |

## 检测当前安装方式

更新前必须先检测出当前方式，然后沿用它。

第一步，看真实路径（启动器是软链，直接看 `which` 的结果会误判）：

```bash
readlink -f "$(command -v claude)"
```

**注意**：npm / pnpm 装出来的命令在 PATH 上可能是 shim *脚本*，`readlink -f` 拿到的还是 shim 自己，
这时从 shim 里挖真实路径，或者直接查包管理器的全局列表：

```bash
grep -o 'cmd-shim-target=.*' "$(readlink -f "$(command -v <命令>)")"
```

第二步，按**路径片段**对照上表（用片段而不是完整目录，默认位置可能被配置改掉）：

| 真实路径片段 | 安装方式 |
| --- | --- |
| `claude/versions/` | 原生安装脚本（默认 `~/.local/share/claude/versions/`） |
| `node_modules/@anthropic-ai/claude-code/` | npm 系，继续第三步 |
| macOS 上在 `/opt/homebrew` 或 `/usr/local/Caskroom/` 下 | brew |
| 其它 | winget / 系统包管理器 / 手动放二进制 |

第三步，区分是哪个包管理器：

```bash
npm ls -g --depth=0 2>/dev/null | grep "@anthropic-ai/claude-code"
pnpm ls -g --depth=0 2>/dev/null | grep "@anthropic-ai/claude-code"
bun pm ls -g 2>/dev/null | grep "@anthropic-ai/claude-code"
brew list --cask 2>/dev/null | grep claude
```

哪个命中就是哪种方式。多个都命中说明装了多份，用 `which -a claude` 看生效的是哪个。

原生安装的话可以再确认一下启动器状态：

```bash
ls -l ~/.local/bin/claude
ls ~/.local/share/claude/versions/
```

`~/.local/bin/claude` 应该是指向 `~/.local/share/claude/versions/<版本>` 的软链。
`versions/` 下会同时留着多个版本，原生安装器只保留最近的两个。

## 更新

**沿用检测出来的安装方式，不要换方式。** 换方式的结果是同一个命令名出现两份、PATH 上互相覆盖，
或者升级命令去改了另一个包管理器的目录。

自带的更新命令会沿用原方式更新：

```bash
claude update
```

它检测不出来或和上面检测结果不一致时，按「安装方式清单」里的「沿用该方式的更新命令」手动执行。

注意各方式的自动更新行为不一样：

- 原生安装：启动时后台自动更新，`claude update` 是手动触发同一套流程。
- brew / winget / apt / dnf / apk：不自动更新，必须用各自的包管理器命令。
- npm 系：claude 自己会尝试自更新，前提是 npm 全局目录可写。

### 发布通道

原生安装、npm、apt/dnf/apk 走 `settings.json` 里的 `autoUpdatesChannel`：

```json
{
  "autoUpdatesChannel": "latest"
}
```

`latest` 是默认值，`stable` 落后一周左右。brew 不看这个设置，用 cask 名区分：
`claude-code` 是 stable，`claude-code@latest` 是 latest。

## 注意

- 把 `~/.local/bin/claude` 换成自己的脚本或软链后，原生安装器就不再管这个启动器了：
  新版本照样装到 `versions/` 下，但跑哪个版本由你的启动器决定，而且所有历史版本都会被保留。
  `claude doctor` 会报出这种情况。想交回给原生安装器管理，删掉启动器再跑 `claude update`。
- npm 方式升级要用 `npm install -g @anthropic-ai/claude-code@latest`，不要用 `npm update -g`——
  后者受原安装时的 semver 范围限制，可能升不到最新。
- npm 方式不要用 `sudo npm install -g`，会导致自动更新失败。
- 原生安装（以及 npm 安装）可以用 `settings.json` 的 `env.DISABLE_AUTOUPDATER = "1"` 关掉自动更新。
- 升级前退出正在运行的 claude 会话。

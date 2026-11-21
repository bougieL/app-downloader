# 系统字体（Maple Mono Normal CN）

本机统一的字体方案：**等宽用 Maple Mono Normal CN，中文字形也用同一个字体**。

## 为什么是它

| 需求 | 结论 |
| --- | --- |
| 拉丁等宽、终端观感好 | Maple Mono 是 JetBrains Mono 衍生，保留斜杠零，笔画像素级均匀 |
| 中文字形不细、不糊 | 自带中文字形（CJK 家族基于 Resource Han Rounded），不走 fallback |
| 中英对齐 | 全角 = 半角 × 2，终端表格和 Markdown 表格不错位 |
| 拉丁中文同一字重 | 同一家族出两套字形，不会出现「英文粗、中文细」 |

对比的其他方案：

- **Noto Sans/Serif/Mono CJK SC**：中文字形没问题，但拉丁部分偏宽偏软，不是等宽手感，
  当终端字体观感一般。
- **Adwaita Sans / Adwaita Mono**：GNOME 自带，拉丁好看，但**完全不含中文字形**，
  中文靠逐字回退到 Droid Sans Fallback —— 那是个 bitmap 兜底字体，笔画细、边缘糊。
- **JetBrains Mono / Source Code Pro / Fira Code 等纯拉丁等宽**：拉丁很好，
  但中文同样要靠 fallback，还是细。

## 安装

Maple Mono 不在 Fedora 仓库里，从 GitHub Releases 下 zip（不用 7z）：

```bash
# 认得「CN」的资产才是带中文字形的版本
curl -LO https://github.com/subframe7536/maple-font/releases/download/v7.9/MapleMono-CN-unhinted.zip
unzip -q MapleMono-CN-unhinted.zip -d extracted
mkdir -p ~/.local/share/fonts/maple-mono-cn
cp extracted/*.ttf ~/.local/share/fonts/maple-mono-cn/
fc-cache -f
```

资产命名规律（`MapleMono-CN-unhinted.zip` 为例）：

| 片段 | 含义 |
| --- | --- |
| `MapleMono` / `MapleMonoNL` | NL = 不带编程连字（No Ligatures） |
| `CN` | 带中文字形（**没有这个就没有中文**） |
| `NF` | 带 Nerd Font 图标 |
| `unhinted` | 不做 hinting，终端里更清晰 |

装到 `~/.local/share/fonts/` 而不是 `/usr/share/fonts/`：不需要 sudo，
`fc-cache` 刷新后所有应用立即可用。

**注意**：新装的字体需要**重启对应应用**才会被识别。
Herdr App 只在启动时扫描字体目录，改完 `settings.json` 不重启的话新字体不生效。

### 资产名和家族名都改过（2026-11 实测）

老版本的资产叫 `MapleMonoNormal-CN-unhinted.zip`，字体家族名是
`Maple Mono Normal CN`（Normal 是字形风格的名字）。**v7.9 起这套命名改了**：

| | 老版本 | 新版本（v7.9+） |
| --- | --- | --- |
| 资产名 | `MapleMonoNormal-CN-unhinted.zip` | `MapleMono-CN-unhinted.zip` |
| 家族名 | `Maple Mono Normal CN` | `Maple Mono CN` |
| 圆角风格 | `MapleMonoRound-CN-...` | `MapleMonoRounded-CN-...` |

所以老配置里的字体名在新版字体上**匹配不上**，`fc-match "Maple Mono Normal CN"`
会回退到别的字体，看起来像「字体没装上」。核对家族名：

```bash
fc-list --format "%{family}\n" ~/.local/share/fonts/maple-mono-cn/*.ttf | tr ',' '\n' | sort -u
```

装完确认：

```bash
fc-match "Maple Mono CN"     # 应命中 MapleMono-CN-Regular.ttf
```

## 各应用的设置

### Herdr App

配置在 `~/.config/herdr-app/settings.json`（改完即时生效，不用重启）：

```json
{
  "font": "Maple Mono CN",
  "font_size": 14,
  "font_weight": 400,
  "ui_font": "Maple Mono CN"
}
```

- `font` / `font_size` / `font_weight` 只在布局没自己指定时才生效；
  `main` 布局不指定，用 `settings.json` 的；`terminals` 布局只定 `font_size: 15`
  （字体仍继承 `settings.json`）；`win98` 布局自带像素字体，不受影响。
- `ui_font` 是面板（mods）的字体，本机也设成 Maple Mono CN，界面和终端统一。
- `font_weight` 默认 500（Medium），400 是 Regular。
- `font_ligatures` 默认关闭，Maple Mono 带连字那套（`MapleMono-CN-*`，
  不带连字的是 `MapleMonoNL-CN-*`）可以设 `true` 开启。
- Herdr **没有单独的中文字体设置项**，终端字体只能整体指定。

### Zed

配置在 `~/.config/zed/settings.json`：

```json
{
  "ui_font_family": "Maple Mono CN",
  "buffer_font_family": "Maple Mono CN",
  "terminal": {
    "font_family": "Maple Mono CN"
  }
}
```

`terminal.font_family` 不设时默认继承 `buffer_font_family`。
Zed 用 cosmic-text 渲染，自己管字体回退，**不读 fontconfig 的中文规则**，
所以必须逐个设置项写明。

### 输入法候选词（ibus）

GNOME Shell 的候选窗口由主题 CSS 写死了 `font-family: monospace`
（在 `gnome-shell-theme.gresource` 的 `candidate-popup-content` 规则里），
所以候选词走的是 fontconfig 的 `monospace` 通用族，**不跟随界面字体**。

因此只要让 fontconfig 的 `monospace` 解析到 Maple，候选词就跟着变了。

## fontconfig：把中文回退指向 Maple

配置在 `~/.config/fontconfig/fonts.conf`：

```xml
<?xml version="1.0"?>
<!DOCTYPE fontconfig SYSTEM "fonts.dtd">
<fontconfig>
  <match target="font">
    <test name="family"><string>Noto Sans CJK SC</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Maple Mono CN</string></edit>
  </match>
  <match target="font">
    <test name="family"><string>Noto Sans Mono CJK SC</string></test>
    <edit name="family" mode="assign" binding="strong"><string>Maple Mono CN</string></edit>
  </match>
</fontconfig>
```

做法是**把字体库里的 CJK 字体改名成 Maple Mono CN**，于是任何「主字体缺中文字形」
的逐字回退都落到 Maple 上，而拉丁字母仍由各自主字体渲染。

改完 `fc-cache -f` 刷新。

### 不要用 mode="append" 追加 Maple

试过用

```xml
<match target="pattern">
  <test name="family"><string>Adwaita Sans</string></test>
  <edit name="family" mode="append" binding="strong"><string>Maple Mono CN</string></edit>
</match>
```

不行。`append` 会让 `sans-serif` / `monospace` 这类通用族也优先命中 Maple，
**拉丁字母跟着变成等宽**，整个桌面观感全变（GNOME 界面、菜单、应用标题都变）。
改字体家族名（`target="font"` + `mode="assign"`）才是只影响缺字回退的做法。

### 副作用：通用族也跟着变成 Maple（2026-11 实测）

`assign` 改的是**家族名**，不是「加一个 fallback」。原来的 CJK 字体是
`sans-serif` / `monospace` 通用族的首选候选，改名之后这两个通用族的首选就变成了
Maple：

```
改动前： fc-match sans-serif → "Noto Sans CJK SC"
改动后： fc-match sans-serif → "Maple Mono CN"
```

`monospace` 变 Maple 是**想要的**（终端、ibus 候选词都走它）。
`sans-serif` 也跟着变，意味着**直接写 `font-family: sans-serif` 的界面，拉丁字母会变等宽**。
GNOME Shell 自己写的是 `Adwaita Sans`，不受影响；影响的是那些直接用通用族的应用。

想只动 `monospace`、不动 `sans-serif`，就只保留 `Noto Sans Mono CJK SC` 那一条规则，
删掉 `Noto Sans CJK SC` 那条。

对比检查（临时移走用户配置再测，`FONTCONFIG_FILE=` 挡不住用户配置，见下）：

```bash
mv ~/.config/fontconfig/fonts.conf /tmp/fonts.conf.bak && fc-cache -f
fc-match sans-serif; fc-match monospace
mv /tmp/fonts.conf.bak ~/.config/fontconfig/fonts.conf && fc-cache -f
fc-match sans-serif; fc-match monospace
```

## 验证

`fc-match` 只看通用族解析，**看不出逐字回退**，要验这个得用 Pango：

```python
import gi
gi.require_version('Pango', '1.0')
gi.require_version('PangoCairo', '1.0')
from gi.repository import Pango, PangoCairo
import cairo

for fam in ['Adwaita Sans', 'monospace', 'sans-serif']:
    s = cairo.ImageSurface(cairo.FORMAT_A8, 300, 50)
    c = cairo.Context(s)
    layout = PangoCairo.create_layout(c)
    layout.set_font_description(Pango.FontDescription(f'{fam} 14'))
    layout.set_text('中文测试', -1)
    it = layout.get_iter()
    while True:
        run = it.get_run_readonly()
        if run is None:
            break
        if run.item is not None:
            print(fam, '->', run.item.analysis.font.describe().get_family())
        if not it.next_run():
            break
```

期望输出：中文那一行是 `Maple Mono CN`，英文那行保持原字体（`Adwaita Sans` 等）。

**测的时候要注意**：临时改 fontconfig 做对比实验时，用户配置
`~/.config/fontconfig/fonts.conf` 会被 `/etc/fonts/fonts.conf` include 进来，
用 `FONTCONFIG_FILE=` 指向临时文件**挡不住它**，会污染对照结果。
正确做法是把用户配置临时移走再测。

## 顺带装的

```bash
sudo dnf install -y jetbrains-mono-fonts
```

Ghostty 的配置里没有 `font-family`，它用 fontconfig 的 `monospace` 默认值；
系统里本来没有 JetBrains Mono，装上之后 Ghostty 的拉丁字形才是 JetBrains Mono。

## 回退

```bash
rm -f ~/.config/fontconfig/fonts.conf && fc-cache -f
rm -rf ~/.local/share/fonts/maple-mono-cn && fc-cache -f
```

再把各应用配置里的 `font` / `font_family` 字段删掉即可。

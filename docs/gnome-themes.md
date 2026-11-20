# GNOME 换主题（结论：别换）

2026-11-20/21 在 Fedora 45（GNOME Shell 51，Wayland）上试过三套主流主题
（Lavanda / WhiteSur / Orchis），全部装完又全部卸载。**本机最终保持 GNOME 原版外观**：
没有 `~/.themes/`，User Themes 扩展已删除，`gtk-theme` = `Adwaita`，`color-scheme` = `default`。

这页记录的是**机制**和**踩过的坑**，下次真要试的时候不用从头踩一遍。

## 换主题的机制

### User Themes 扩展只做一件事

把 `~/.themes/<主题名>/gnome-shell/gnome-shell.css` 交给 shell 的 `Main.setThemeStylesheet()`，
shell 用它当 `application_stylesheet` 加载。

- 扩展扫描的目录（`util.js` 的 `getThemeDirs()`）：`~/.themes`、`~/.local/share/themes`、`/usr/share/themes`
- 文件名是写死的 `gnome-shell/gnome-shell.css`。**扩展不支持 `gnome-shell-dark.css` 这类变体**，
  明暗必须是**不同的主题目录**（`WhiteSur-Light` / `WhiteSur-Dark`）
- 主题名 = `~/.themes/` 下的目录名

### GNOME 45+ 的 shell 是两层 CSS（关键）

`St.Theme` 同时加载两张表：

| 层 | 内容 | 优先级 |
| --- | --- | --- |
| 基础层 `default_stylesheet` | `gnome-shell-dark.css` / `gnome-shell-light.css`（系统自带） | 低 |
| 用户层 `application_stylesheet` | User Themes 给的 `gnome-shell.css` | 高 |

基础层由 `org.gnome.desktop.interface color-scheme` 决定走哪个变体，逻辑在 shell 的
`js/ui/main.js`：

```js
export function getStyleVariant() {
    const {colorScheme} = St.Settings.get();
    switch (sessionMode.colorScheme) {
    case 'prefer-dark':
        return colorScheme === St.SystemColorScheme.PREFER_LIGHT ? 'light' : 'dark';
    ...
    default:
        return '';
    }
}
```

`sessionMode.colorScheme` 默认是 **`prefer-dark`**（`js/ui/sessionMode.js` 里
`DEFAULT_MODE = 'restrictive'` 定的，`user` 模式继承它）。所以：

- **不设 `color-scheme`，基础层永远是 dark**
- 只有 `color-scheme = 'prefer-light'` 才会切到 `gnome-shell-light.css`
- 改完立即重载：`St.Settings.get().connect('notify::color-scheme', _loadDefaultStylesheet)`

用户层优先级更高，但它**只覆盖它自己写到的选择器**（面板、强调色、按钮之类），
概览背景、快速设置、OSD 这些大头仍然来自基础层。

**结论：换主题必须「主题目录的明暗变体 + `color-scheme`」配套改。**
只切主题目录的话，基础层还是黑的，整体看起来几乎没变——这是「换了跟没换一样」的第一层原因。

```bash
gsettings set org.gnome.desktop.interface color-scheme 'prefer-light'   # 基础层切浅色
gsettings set org.gnome.desktop.interface color-scheme 'prefer-dark'    # 基础层切深色
```

### 判断 shell 当前到底加载了哪套样式

```bash
journalctl --user -b | grep -i "user theme"
# loading user theme: ~/.themes/WhiteSur-Dark/gnome-shell/gnome-shell.css
# loading default theme (Adwaita)      ← 扩展没启用时是这行
```

## 坑

### User Themes 扩展会被悄悄关掉

**现象**：切主题完全没反应，界面还一直是暗色；日志里有

```
JS ERROR: TypeError: can't access property "get_string", this._settings is null
```

**原因**：扩展被 disable 后 `_settings` 置 null、shell 退回默认样式；
之后任何人再改主题名，就会走到 `this._settings.get_string()` 抛上面那条错。
本机这次就是扩展被扩展管理器（`com.mattjakeman.ExtensionManager`）同步 `enabled-extensions`
时漏掉了，于是**所有主题看起来都一样**。

**排查**（先做这一步，能省掉一半时间）：

```bash
gnome-extensions info user-theme@gnome-shell-extensions.gcampax.github.com | grep -E "已启用|状态"
gsettings get org.gnome.shell enabled-extensions | tr ',' '\n' | grep user-theme
```

重新启用：

```bash
gnome-extensions enable user-theme@gnome-shell-extensions.gcampax.github.com
```

### gsettings 改主题名要带 schema 路径

该扩展的 gschema 只编译在扩展目录里（`.../user-theme@.../schemas/gschemas.compiled`），
普通写法会报「没有 org.gnome.shell.extensions.user-theme 架构」：

```bash
ext=~/.local/share/gnome-shell/extensions/user-theme@gnome-shell-extensions.gcampax.github.com
GSETTINGS_SCHEMA_DIR=$ext/schemas gsettings set org.gnome.shell.extensions.user-theme name 'WhiteSur-Dark'
```

### vinceliuice 系安装脚本：TERM 没设就静默退出

WhiteSur / Orchis 的 `install.sh` 用 `tput` 上色。**在 TERM 未设置的终端里跑**（比如某些
非交互环境、`cron`、CI），`tput` 失败触发 ERR trap，脚本以 exit 1 静默退出，
而且它把 stderr 重定向进了 `/tmp/WhiteSur.lock/error_log.txt`，退出时的 trap 又把
`/tmp/WhiteSur.lock/` 整个删掉——**连错误日志都看不到**，现象就是「跑完了，什么也没发生」。

```bash
TERM=xterm-256color bash install.sh -d ~/.themes
```

其他相关：

- `--silent-mode` 要求 root，普通用户用不了（它就是给「跳过确认」用的）
- 默认安装目标是 `/usr/share/themes`（系统级，要 sudo），装到用户目录要显式 `-d ~/.themes`
- 装完顺带往 `~/.local/bin/` 塞一个 `gnome-theme-switcher`（GTK4 小工具，**只切 GTK 主题，不切 shell**）

### Orchis 的 GTK4 样式表是坏的

`~/.themes/Orchis-Dark/gtk-4.0/gtk.css` 里有 **45 处未编译的 Sass 变量**（`--blue-1: $blue_1;`）
和 **107 处 GTK 不支持的 CSS `var()`**（`var(--accent-bg-color)`）→ GTK4 应用直接报：

```
Theme parser error: gtk.css:8672:39-43: Expected a valid color.
```

受影响的正是 modern 应用：设置、文件(Nautilus)、文本编辑器、截图工具——
表现是**样式残缺 + 每次启动刷一屏报错**。Orchis 的 `gtk-3.0/gtk.css` 是干净的，
所以只有 GTK4 那半边坏。

WhiteSur 的 `gtk-4.0/gtk.css` 也有类似问题，但轻一些：用了 GTK4 不支持的 CSS `alpha()`
函数（`alpha(rgba(0,0,0,0.08),0.7)`），报同样的 parse error，
不过只影响装饰性渐变（侧边栏阴影之类），不至于整块样式崩掉。

**验一个主题的 GTK4 样式表干不干净**：

```bash
grep -cE '\$[a-zA-Z0-9_-]+' ~/.themes/<主题>/gtk-4.0/gtk.css   # Sass 变量残留，应为 0
grep -cE 'var\(--' ~/.themes/<主题>/gtk-4.0/gtk.css              # GTK 不支持，应为 0
grep -cE 'alpha\(' ~/.themes/<主题>/gtk-4.0/gtk.css              # GTK4 不支持，非 0 会有报错
```

### libadwaita 应用要走 `~/.config/gtk-4.0/`

GTK4/libadwaita 应用不认 `gtk-theme` 里的 gtk-4.0 目录，要软链到用户配置目录：

```bash
mkdir -p ~/.config/gtk-4.0
cd ~/.config/gtk-4.0
ln -sfn ~/.themes/<主题>/gtk-4.0/assets     assets
ln -sfn ~/.themes/<主题>/gtk-4.0/gtk.css    gtk.css
ln -sfn ~/.themes/<主题>/gtk-4.0/gtk-dark.css gtk-dark.css
```

注意 Orchis 的安装器 `-l` 选项连着装了 Dark 和 Light 两个变体，**最后链接的是 Light**，
得自己改回来。

## 装过的三套

| 主题 | 仓库 | 特点 | 问题 |
| --- | --- | --- | --- |
| Lavanda | `WenYin-Community/Lavanda-gtk-theme` | 唯一原生支持 GNOME 51 的：仓库有 `src/sass/gnome-shell/widgets-51-0/`，`install.sh` 读 `gnome-shell --version` 自动切分支 | README 里的 Copr 源 `ruojiner/lavanda-kde-theme` **已 404**，只能手动装；`install.sh -c` 的变体值是**小写** `standard\|light\|dark`，传 `Dark` 直接报错退出 |
| WhiteSur | `vinceliuice/WhiteSur-gtk-theme` | 最热门（★9000+），macOS Big Sur 风 | shell 样式分支只到 `widgets-48-0`，GNOME 51 上降级用 48 的样式（能显示，不带 51 新特性）；安装脚本的 TERM 坑 |
| Orchis | `vinceliuice/Orchis-theme` | Material 圆角风，★4000+ | **GTK4 样式表坏**（见上） |

装主题共同需要 `sassc`（编译 scss）；WhiteSur 还要 `glib2-devel`。

## 卸载 / 还原

```bash
# 1. 还原设置
gsettings set org.gnome.desktop.interface gtk-theme 'Adwaita'
gsettings set org.gnome.desktop.interface color-scheme 'default'

# 2. 禁用并删除扩展
gnome-extensions disable user-theme@gnome-shell-extensions.gcampax.github.com
rm -rf ~/.local/share/gnome-shell/extensions/user-theme@gnome-shell-extensions.gcampax.github.com

# 3. 删主题和链接
rm -rf ~/.themes ~/.local/share/themes ~/.config/gtk-4.0

# 4. 删安装器顺带塞的东西
rm -f ~/.local/bin/gnome-theme-switcher
rm -f ~/.local/share/applications/org.gnome.GTK4ThemeSwitcher.desktop

# 5. 卸只为编译主题装的包
sudo dnf remove sassc glib2-devel
```

第 3 步注意：`~/.themes/`、`~/.config/gtk-4.0/` 都是装主题时新建的，
删之前先 `ls` 确认里面没有自己放的东西。

## 结论

GNOME 45+ 之后换主题的投入产出比很低：

- 明暗由 `color-scheme` 决定，不是由主题决定，配错了就是「换了跟没换一样」
- 热门主题的 shell 分支普遍落后 GNOME 51 一到三个版本
- 主流主题的 GTK4 样式表普遍有解析错误，modern 应用样式残缺
- 第三方扩展还会被扩展管理器悄悄关掉

保持原版 Adwaita 是最省心的选择。真要改外观，优先动 **icon theme / cursor / wallpaper**
（不影响 shell 样式，出错面小），而不是 shell 主题。

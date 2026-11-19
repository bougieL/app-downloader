# 中文输入法（ibus-libpinyin）

Fedora GNOME（Wayland）下的中文拼音输入法。GNOME 原生使用 IBus 框架，
拼音引擎是 `ibus-libpinyin`（Intelligent Pinyin），系统包里已默认带好。

## 安装方式清单

| 方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 系统包管理器（dnf） | `rpm -q ibus-libpinyin` 能查到 | `sudo dnf upgrade ibus-libpinyin` |
| Flatpak（org.fcitx.Fcitx5 等另一套框架） | `flatpak list` 里有 Fcitx5 | `flatpak update` |
| 源码编译（`meson` + `ninja`） | 自建目录里有 `build.ninja` | 重新 `ninja && ninja install` |

## 检测当前安装方式

```bash
rpm -q ibus-libpinyin          # dnf 装的
flatpak list | grep -i fcitx    # flatpak 装的另一套框架
```

判断不出来问用户，不要猜，也不要擅自换方式重装。

## 安装（dnf）

```bash
sudo dnf install -y ibus-libpinyin
```

装完在 GNOME 设置里确认：

```bash
gsettings get org.gnome.desktop.input-sources sources
# 期望输出里同时有 ('xkb', 'us') 和 ('ibus', 'libpinyin')
```

没有的话在「设置 → 键盘 → 输入源 → 添加输入源 → 中文（智能拼音）」里加。

## 使用

- 切换中英文：`Super + Space`（反向切换 `Shift + Super + Space`）。
- 查看当前引擎：`ibus engine`（返回 `libpinyin` 或 `xkb:us::eng`）。
- 手动切到拼音：`ibus engine libpinyin`。
- 引擎配置（在「设置 → 键盘 → 输入源 → 智能拼音 → 齿轮」里改，也可以直接跑
  `ibus-setup-libpinyin`）。

Qt 程序需要 `QT_IM_MODULE=ibus`、`XMODIFIERS=@im=ibus`，Fedora 默认已经带上了；
GTK4 程序走 ibus portal（`ibus-portal`），不需要额外配置。

## 候选词字体

**候选词窗口不跟随界面字体。** GNOME Shell 的主题 CSS 里写死了
`font-family: monospace`（在 `gnome-shell-theme.gresource` 的
`candidate-popup-content` 规则里），所以候选词走的是 fontconfig 的 `monospace` 通用族。

要改候选词字体，改 fontconfig 让 `monospace` 解析到想要的字体就行。
本机用 `~/.config/fontconfig/fonts.conf` 把 CJK 回退指向 Maple Mono Normal CN，
候选词的中文就跟着变了。配置和原理见 [docs/fonts.md](fonts.md)。

## 更新

先按上面的方法检测安装方式，再沿用：

```bash
rpm -q ibus-libpinyin && sudo dnf upgrade ibus-libpinyin
```

`ibus-libpinyin` 升级时依赖的 `libpinyin-data`（词库）会一起更新。

## 备选：换用 fcitx5

只有在 ibus 在某个应用里不好使时才换。fcitx5 是另一套输入法框架，
在 GNOME Wayland 下需要把 IBus 换成 fcitx5：

```bash
sudo dnf install -y fcitx5 fcitx5-chinese-addons
```

切换步骤：设置 `QT_IM_MODULE=fcitx`、`XMODIFIERS=@im=fcitx`，GNOME「输入源」里
删掉ibus 项并添加 fcitx5，自启里启用 `fcitx5-autostart`。**换框架要重登 Wayland 会话。**

## 注意

- **ibus 的 D-Bus 接口不在会话总线上。** GNOME 45 的 ibus-daemon（1.5.35）在
  `~/.cache/ibus/dbus-XXXXXX` 上开了一个**私有**总线，`org.freedesktop.IBus` 这个
  well-known name 只存在于这条私有总线上。在会话总线上调
  `org.freedesktop.IBus.*` 会报「对象在路径 /org/freedesktop/IBus 处不存在」。
  调试要用私有总线：

  ```bash
  ibus address            # unix:path=~/.cache/ibus/dbus-XXXXXX,guid=...
  ADDR=$(ibus address | cut -d, -f1)
  gdbus introspect --address "$ADDR" --dest org.freedesktop.IBus \
    --object-path /org/freedesktop/IBus --xml
  ```

- **私有总线上的接口跟老文档不一样。** 新版没有 `SetCurrentEngine` /
  `GetCurrentEngineName`，改成了 `SetGlobalEngine` / `GetGlobalEngine`；
  `CreateInputContext` 只收一个 `client_name`；`ProcessKeyEvent` 去掉了 `is_press`
  参数（只有 keyval / keycode / state）；输入上下文也不支持 `FocusIn`
  （报 "The input context does not support focus"）。写脚本调 ibus 时别照搬老接口。

- **ibus 开了全局引擎时不能单独设引擎。** 调 `SetEngine` 会报
  "Cannot set engines when use-global-engine is enabled."，此时用
  `ibus engine <名字>` 切全局引擎。

- **mutter 不支持虚拟键盘协议。** `wtype` 之类靠
  `zwp_virtual_keyboard_manager_v1` 注入按键的工具在 GNOME 下会直接报
  "Compositor does not support the virtual keyboard protocol"，所以没法用脚本
  自动模拟输入做端到端测试，只能手工验证。

- **`ibus-libpinyin` 只是引擎，词库在 `libpinyin-data` 里**（`/usr/lib64/libpinyin/data`），
  没装的话拼音出不来候选词。

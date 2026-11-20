# 中文输入法（ibus-rime + 雾凇拼音）

Fedora GNOME（Wayland）下的中文拼音输入法。GNOME 原生用 IBus 框架，本机现在跑的是
**ibus-rime 引擎 + 雾凇拼音方案（rime-ice）**，词库和语法模型都比原来的 `ibus-libpinyin`
强得多（见下面「学到了什么」）。

## 现状

| 部件 | 位置 | 说明 |
| --- | --- | --- |
| 引擎 | `ibus-rime`（系统包） | Rime 引擎，共享数据在 `/usr/share/rime-data` |
| 方案 | `~/.config/ibus/rime/`（雾凇拼音 rime-ice） | 用户目录，190 万词条 |
| 语法模型 | `~/.config/ibus/rime/wanxiang-lts-zh-hans.gram` | 万象 LTS，391 MB |
| 插件 | `librime-octagram` / `librime-lua` / `librime-predict` | 系统包，装在 `/usr/lib64/rime-plugins/` |
| 用户词频 | `~/.config/ibus/rime/rime_ice.userdb` | 自动学习，leveldb 格式 |

`ibus-libpinyin` 仍然装着，作为备选引擎保留在 GNOME 输入源列表里。

## 安装方式清单

分两层装，别混：

| 层 | 方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- | --- |
| 引擎 + 插件 | 系统包管理器（dnf） | `rpm -q ibus-rime` 能查到 | `sudo dnf upgrade ibus-rime librime-octagram librime-lua librime-predict` |
| 输入方案 + 语法模型 | plum（`~/plum`） | `~/plum` 目录存在，`~/.config/ibus/rime/` 里有 `default.yaml` | 见下面「更新」 |

方案文件本身来自 GitHub（`iDvel/rime-ice`），但**不走 GitHub Release 下载安装包**，
走 plum 的配方机制。

## 检测当前安装方式

```bash
rpm -q ibus-rime librime-octagram    # 引擎层
ls ~/plum/rime-install                 # 方案层
ls ~/.config/ibus/rime/default.yaml    # 已部署的方案
```

判断不出来问用户，不要猜，也不要擅自换方式重装。

## 安装（dnf）

```bash
sudo dnf install -y ibus-rime librime-octagram librime-lua librime-predict librime-tools
```

`librime-tools` 提供 `rime_deployer`（重新部署）、`rime_dict_manager`（词典管理）、
`rime_table_decompiler`（词表反编译，查词库内容用）。

三个插件都是**运行时自动加载**的，librime 会扫 `/usr/lib64/rime-plugins/`，
不用改配置。确认是否加载：

```bash
grep -E "octagram|lua|predict" /proc/$(pgrep -f ibus-engine-rime | head -1)/maps | awk '{print $6}' | sort -u
```

## 安装（雾凇拼音方案）

```bash
git clone --depth 1 https://github.com/rime/plum.git ~/plum
cd ~/plum
rime_dir="$HOME/.config/ibus/rime" bash rime-install iDvel/rime-ice
```

再装语法模型（万象 LTS，约 391 MB，GitHub Releases 下载，国内大概 10 分钟）：

```bash
cd ~/plum
rime_dir="$HOME/.config/ibus/rime" bash rime-install iDvel/rime-ice:others/recipes/grammar:schema=rime_ice
```

配方做的事：下载 `wanxiang-lts-zh-hans.gram` 到用户目录，并生成 `rime_ice.custom.yaml`：

```yaml
__patch:
  - patch/+:
      grammar:
        language: wanxiang-lts-zh-hans
        collocation_max_length: 6
        collocation_min_length: 3
        collocation_penalty: -14
        non_collocation_penalty: -6
        weak_collocation_penalty: -100
        rear_penalty: -20
      translator/contextual_suggestions: false
      translator/max_homophones: 8
```

## 加入 GNOME 输入源

```bash
gsettings set org.gnome.desktop.input-sources sources \
  "[('ibus', 'rime'), ('ibus', 'libpinyin'), ('xkb', 'us')]"
ibus write-cache      # 刷新引擎缓存，否则新引擎不出现在 ibus list-engine 里
ibus restart          # 重启 ibus 守护进程
ibus engine rime      # 立刻切过去
```

`ibus write-cache` 会打印一堆 "Engines exec:... is failed" 的警告，是正常的——
那些引擎在 `--xml` 模式下退出方式特殊，不影响 rime。

## 目录与文件

```
~/.config/ibus/rime/
├── default.yaml            # 雾凇拼音原版配置（会被更新覆盖）
├── default.custom.yaml     # ★ 全局自定义补丁（不会被覆盖）
├── rime_ice.schema.yaml    # 雾凇拼音全拼方案原版
├── rime_ice.custom.yaml    # ★ 方案级补丁（语法模型参数在这里）
├── rime_ice.dict.yaml      # 词库挂载表（import_tables）
├── cn_dicts/               # 基础/扩展/腾讯词向量等分卷
├── lua/                    # 雾凇的 Lua 扩展（查词、拆字、日期、英文派生等）
├── custom_phrase.txt       # 自定义短语
├── wanxiang-lts-zh-hans.gram  # 万象语法模型
├── build/                  # 编译产物（74 MB），rime_deployer 生成
└── rime_ice.userdb/        # 用户词频（自动学习）
```

## 可配置项

### 方案选单里切（F4 或 `Ctrl + \``）

这 6 个开关在方案里定义，不用改文件：

| 开关 | 状态 | 说明 |
| --- | --- | --- |
| `ascii_mode` | 中 / Ａ | 中英切换 |
| `ascii_punct` | ¥ / $ | 中英标点 |
| `traditionalization` | 简 / 繁 | 简繁转换（OpenCC，默认香港繁体） |
| `emoji` | 💀 / 😄 | Emoji（默认开） |
| `full_shape` | 半角 / 全角 | 全半角 |
| `search_single_char` | 词 / 单 | 辅码反查时单字优先 |

`default.yaml` 里 `switcher.hotkeys` 决定 F4 / `Ctrl+grave` / `Ctrl+Shift+grave` 呼出，
`switcher.save_options` 决定哪些开关的记忆是跨窗口保留的。

### 改文件（改完要重新部署）

| 想改什么 | 改哪个文件 |
| --- | --- |
| 候选词个数、方案列表、翻页/选字快捷键、标点、网址识别 | `default.custom.yaml`（本机已设 `page_size: 9`、只保留 `rime_ice` 方案） |
| 语法模型的行为（搭配长度、各项惩罚分） | `rime_ice.custom.yaml` 的 `grammar:` 段 |
| 模糊音（n/l、zh/z、an/ang……） | `rime_ice.schema.yaml` 里 `speller/algebra` 下注释掉的 `derive` 规则，取消注释即可 |
| 繁体转换目标（港繁/台繁） | `rime_ice.schema.yaml` 的 `traditionalize/opencc_config` |
| 自定义短语（人名、地址、常用语，固定上屏） | `custom_phrase.txt`，Tab 分隔「词 / 编码 / 权重」 |

重新部署：

```bash
cd ~/.config/ibus/rime && rime_deployer --build .
```

`rime_ice.schema.yaml` 这类原版文件**会被 plum 更新覆盖**，改完记得留个副本，
或者把改动挪到 `.custom.yaml` 补丁里。

## 学到了什么

- **用户词频学习**：默认开着。选过的词会记进 `~/.config/ibus/rime/rime_ice.userdb`，
  之后排序自动往上提。词典里根本不存在的词没法靠学习产生，得走自定义词库。
- **语法学习**：万象语法模型（octagram 插件）负责长句准确率，按「搭配」而非「词频」排序。
  模型参数就是上面 `rime_ice.custom.yaml` 里 `grammar:` 那几项。
- **词库规模**：主词表 190 万条（`rime_ice.table.bin` 约 61 MB），
  覆盖 8105 字 + 基础词库 + 扩展词库 + 腾讯词向量。

## 更新

方案层用 plum（**只更新方案，不动语法模型**）：

```bash
cd ~/plum
rime_dir="$HOME/.config/ibus/rime" bash rime-install iDvel/rime-ice
cd ~/.config/ibus/rime && rime_deployer --build .
```

语法模型单独更新（配方会重新下载 391 MB，并**覆盖 `rime_ice.custom.yaml`**，
覆盖前先备份自己加的补丁）：

```bash
cd ~/plum
rime_dir="$HOME/.config/ibus/rime" bash rime-install iDvel/rime-ice:others/recipes/grammar:schema=rime_ice
```

引擎层：

```bash
sudo dnf upgrade ibus-rime librime-octagram librime-lua librime-predict librime-tools
```

## 踩坑

- **新引擎不出现要先 `ibus write-cache`。** 装完 `ibus-rime` 后 `ibus list-engine`
  里没有 rime，是守护进程的组件缓存没刷新，`ibus write-cache` + `ibus restart` 才行。

- **别用 `rime_patch`。** 它会**就地改写**目标 `.custom.yaml`：把配方生成的
  `__patch` 结构压成一行、删掉 `# Rx:` 注释、还塞进一段 `customization:` 元数据。
  实测把 `rime_ice.custom.yaml` 整个重写了一遍。只读配置用 `rime_patch <config_id> <key>`。

- **自定义词库挂不上去。** 想在 `import_tables` 里加自己的词库，试过在
  `rime_ice.custom.yaml` 写 `"dictionary/import_tables/+": [mydict]`、以及建
  `rime_ice.dict.custom.yaml` 写 `"import_tables/+": [mydict]`，两种写法
  **重新部署后词条都不进表**（用 `rime_table_decompiler` 反编译 `build/rime_ice.table.bin`
  查过，190 万条里没有自定义词条）。`import_tables` 只在**词典自己的配置**里被读
  （librime `src/rime/dict/dict_settings.cc` 的 `DictSettings::GetTables()`），
  方案级补丁够不着它。要挂自定义词库，只能直接改 `rime_ice.dict.yaml` 的
  `import_tables:`（代价：plum 更新会覆盖这一行）。

- **语法模型下载很慢。** 万象 LTS 是 391 MB 的 GitHub Release 资产，实测约 0.5 MB/s，
  要 12 分钟。plum 用 `curl -fRL` 下载，中断了直接重跑配方即可（已下完的文件会跳过）。

- **重新部署要一两分钟。** 词表 190 万条 + 391 MB 语法模型，
  `rime_deployer --build .` 实测 1～2 分钟，别在打字过程中跑。

- **候选词字体走 monospace。** 和 `ibus-libpinyin` 一样，GNOME Shell 主题 CSS 里
  写死了 `font-family: monospace`，改 fontconfig 即可，见 [docs/fonts.md](fonts.md)。

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

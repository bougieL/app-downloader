# Obsidian

- 仓库地址：https://github.com/obsidianmd/obsidian-releases
- 官网：https://obsidian.md

基于 Markdown 的本地知识库笔记应用，Electron 桌面客户端。
安装包发布在 `obsidianmd/obsidian-releases` 的 release 里（这个仓库只放安装包，
不是 Obsidian 的源码仓库，源码是闭源的）。

## Release 资产

| 平台 | 资产名 |
| --- | --- |
| Linux x64 | `obsidian_<版本>_amd64.deb` |
| Linux arm64 | `obsidian_<版本>_arm64.deb` |
| macOS | `Obsidian-<版本>.dmg` |
| Windows | `Obsidian-<版本>.exe`（安装器）|

同一个 release 里还有 `Obsidian-<版本>.AppImage`、`obsidian-<版本>.tar.gz`（免安装）、
`obsidian-<版本>.asar.gz`（只含 asar 源码，供打包用）、`.apk`（Android）。

## 安装方式清单

装的时候选了哪种，以后就更新哪种。

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| deb（apt / dpkg） | `dpkg -l` 里有 `obsidian`；`/usr/bin/obsidian` 存在 | 下载新 deb 后 `sudo apt install ./obsidian_<版本>_amd64.deb` |
| AppImage | 家里有 `Obsidian-*.AppImage` 文件 | 下载新 AppImage 覆盖 |
| tar.gz 免安装 | 家里有解压出来的 `Obsidian-*.tar.gz` 目录 | 下载新 tar.gz 整目录替换 |
| macOS dmg | `/Applications/Obsidian.app` | 下载新 dmg 覆盖 `.app` |
| Windows exe | `%LOCALAPPDATA%\Programs\Obsidian\` | 运行新安装器覆盖 |

官方没有 apt / dnf 软件源，deb 装完后 `apt-cache policy obsidian` 只会显示已安装版本、
候选版本仍是同一个（优先级 100 表示本地装的），**apt 不会自动更新它**。

## 检测当前安装方式

```bash
dpkg -l | grep obsidian        # deb
ls ~/Downloads/Obsidian*.AppImage 2>/dev/null   # AppImage
command -v obsidian            # 有输出 → 已装
```

判断不出来就问用户，不要猜。

## 更新

```bash
# deb
gh release download <tag> --repo obsidianmd/obsidian-releases \
  --pattern 'obsidian_<版本>_amd64.deb' --dir tmp/obsidian --clobber
sudo apt install ./tmp/obsidian/obsidian_<版本>_amd64.deb
```

**升级前先退出正在运行的 Obsidian**，否则会锁住 `/opt/Obsidian/` 下的文件导致安装失败。

## 注意

- 笔记数据不在安装目录里，在 `~/.config/obsidian/`（配置、插件）和
  `~/Documents/Obsidian/` 或用户自设的库目录（笔记本体），升级不会动它们。
- 官方 `Obsidian-<版本>.asar.gz` 是解包后的 asar 源码，不是可运行的程序，别当安装包下载。
- 社区插件和主题也可能在更新时被要求重新安装，属正常现象。
- **本机状态（2026-11-10 核实）**：Debian 13 (trixie) / amd64，
  `dpkg -l obsidian` 显示 1.13.4，**官方没有 apt 源**，是手动下 deb 装的
  （`apt-cache policy obsidian` 显示候选版本与已装版本相同、优先级 100）。
  本次从 `obsidianmd/obsidian-releases` 的 `v1.15.0` 下载
  `obsidian_1.15.0_amd64.deb` 用 `sudo apt install ./...deb` 升级成功。

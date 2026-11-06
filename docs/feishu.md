# 飞书（Feishu / Lark）

- 官网下载：<https://www.feishu.cn/api/downloads>
- Flatpak 应用 ID：`cn.feishu.Feishu`
- 官方 CDN：`sf3-cn.feishucdn.com`
- deb 包名：`bytedance-feishu-stable`

Linux 桌面版官方叫 **Feishu**（飞书），海外版叫 Lark，国内版功能更全。

## 安装方式清单

| 安装方式 | 识别特征 | 更新命令 |
| --- | --- | --- |
| Flatpak | `flatpak list --app` 里有 `cn.feishu.Feishu` | `flatpak update cn.feishu.Feishu` |
| deb（apt / dpkg） | `dpkg -l` 里有 `bytedance-feishu-stable`；`/opt/bytedance/feishu` 存在 | 下载新 deb 后 `sudo apt install ./Feishu-linux_x64-<版本>.deb` |
| 官方 tar 包 / 免安装 | 家目录里有解压出来的 `feishu` 目录 | 重新下载解压覆盖 |

### Flatpak（推荐）

```bash
flatpak install flathub cn.feishu.Feishu
flatpak run cn.feishu.Feishu
```

Flathub 上的 manifest 直接拉官方 deb 打包（`Feishu-linux_x64-<版本>.deb`），
app ID 是 `cn.feishu.Feishu`，桌面项命令是 `feishu`。

沙箱权限比较克制（manifest 里只有）：

```
--share=network --share=ipc --socket=x11 --socket=pulseaudio
--device=all --talk-name=org.gnome.ScreenSaver
--talk-name=org.freedesktop.Notifications
--filesystem=xdg-download
```

注意**没有 `--socket=wayland`**，所以在纯 Wayland 会话下要靠 XWayland 跑；
只有 `xdg-download` 被放行，其他目录（文档、桌面等）访问不到，
发文件时存到下载目录即可。

### deb

```bash
curl -LO https://sf3-cn.feishucdn.com/obj/ee-appcenter/fc38d53a/Feishu-linux_x64-<版本>.deb
sudo apt install ./Feishu-linux_x64-<版本>.deb
```

deb 装出来的内容在 `/opt/bytedance/feishu`，桌面项 `/usr/share/applications/bytedance-feishu.desktop`。

## 检测当前安装方式

```bash
flatpak list --app | grep -i feishu   # Flatpak
dpkg -l | grep -i feishu              # deb
ls -d /opt/bytedance/feishu           # deb 目录还在不在
```

判断不出来就问用户，不要猜。

## 更新

```bash
# Flatpak
flatpak update cn.feishu.Feishu

# deb
sudo apt install ./Feishu-linux_x64-<版本>.deb
```

**升级前先退出飞书**，运行中直接装 deb 会因为 `/opt/bytedance/feishu` 下的文件被占用而失败。

官网下载页 <https://www.feishu.cn/api/downloads> 的链接里带版本号，
每次都要从页面重新抓直链（`grep -oE 'https://[^"]*Feishu-linux_x64[^"]*\.deb'`）。

## 从 deb 版迁移到 Flatpak

换安装方式时数据不会自动跟着走。飞书的用户数据全在 Electron 风格的目录里：

| 内容 | deb 版 | Flatpak 版 |
| --- | --- | --- |
| 配置、登录态、聊天缓存 | `~/.config/LarkShell` | `~/.var/app/cn.feishu.Feishu/config/LarkShell` |
| 缓存 | `~/.cache/LarkShell` | `~/.var/app/cn.feishu.Feishu/cache/LarkShell` |

注意目录名是 **`LarkShell`**（不是 Feishu），飞书改名成 Lark 后留下的。

步骤：

```bash
# 1. 先退出飞书
pkill -TERM -f '/opt/bytedance/feishu'

# 2. 装 flatpak 版
flatpak install -y flathub cn.feishu.Feishu

# 3. 先启动一次，让它生成目录结构，然后退出
flatpak run cn.feishu.Feishu
# 窗口出来后退掉

# 4. 搬数据（先跑一次 flatpak 生成目录再拷，否则 cp 报错）
cp -a ~/.config/LarkShell/. ~/.var/app/cn.feishu.Feishu/config/LarkShell/
cp -a ~/.cache/LarkShell/.   ~/.var/app/cn.feishu.Feishu/cache/LarkShell/

# 5. 卸载 deb 版
sudo apt purge bytedance-feishu-stable

# 6. 再启动，确认登录态还在
flatpak run cn.feishu.Feishu
```

搬完确认关键文件在：

```bash
ls ~/.var/app/cn.feishu.Feishu/config/LarkShell/ | grep -E 'logout_token|Last Version'
```

`logout_token` 存在说明登录态跟着过来了。

### 坑

- **必须先退出飞书再搬**。Electron 应用运行时持有 sqlite 和缓存文件，硬拷会拷到写了一半的状态。
- **目录名是 `LarkShell` 不是 `Feishu`**，找错目录会以为没有数据。
- flatpak 的 `cache/LarkShell` 里有个 `com.ebus.lark.nf_ipc.sock` 是 unix socket，
  旧目录里那个不要拷过去，拷了会报「地址已在使用」。清掉：
  `rm -f ~/.var/app/cn.feishu.Feishu/cache/LarkShell/com.ebus.lark.nf_ipc.sock`
- 旧版本 `Last Version` 文件里记的是 Electron 版本号（形如 `131.0.6778.268`），
  不是飞书版本，排查时别当成飞书版本看。
- 迁移后旧目录 `~/.config/LarkShell`（152M）和 `~/.cache/LarkShell`（36M）还在，
  确认新版本一切正常再删。
- flatpak 版启动日志里的 `Failed to load module "canberra-gtk-module"` 是无害警告，
  只是沙箱内没有该模块，不影响使用。

## 注意

- 官方 deb 版本更新很快，flatpak 版通常会跟着一起更新，但会有 1～2 天延迟。
- 飞书沙箱版只放行了 `xdg-download`，要共享文档给外部应用（比如拖到编辑器），
  得先在系统设置里给 `cn.feishu.Feishu` 加 `--filesystem=xdg-documents` 等权限：
  ```bash
  flatpak override --user --filesystem=xdg-documents cn.feishu.Feishu
  ```
  （`--user` 改的是用户级安装；system 安装要用 `sudo flatpak override --system`。）
- 飞书窗口在 Wayland 下不能置顶、无法全局快捷键唤出，用 XWayland 最省事。
- 卸载 deb 版后 `~/下载/Feishu-linux_x64-<版本>.deb` 这种下载的安装包不会自动删，手动清。

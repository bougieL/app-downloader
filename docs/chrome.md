# Google Chrome

- 官网：<https://www.google.com/chrome/>
- 官方 rpm 仓库：`https://dl.google.com/linux/chrome/rpm/stable/x86_64`
- 官方 deb 仓库：`https://dl.google.com/linux/chrome/deb/`
- Flathub 应用 ID：`com.google.Chrome`
- Fedora 上装的是 **Google Chrome 正式版**，不是 Chromium（Chromium 见 `docs/gnome-themes.md` 之外的说明，
  Fedora 官方源的 `chromium` 是另一个包）。

## 安装方式清单

**装的时候选了哪种，以后就更新哪种。**

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方 rpm 仓库（dnf） | `rpm -q google-chrome-stable` 有输出 | `sudo dnf update google-chrome-stable` |
| 官方 deb 仓库（apt） | `dpkg -l google-chrome-stable` 有输出 | `sudo apt update && sudo apt install --only-upgrade google-chrome-stable` |
| Flathub | `flatpak list \| grep com.google.Chrome` | `flatpak update com.google.Chrome` |
| 手动下 rpm 本地装 | `rpm -q google-chrome-stable` 有输出，但仓库文件里 `enabled=0` | 重新下载 rpm 再 `sudo dnf install ./<包名>.rpm` |

## 检测当前安装方式

```bash
rpm -q google-chrome-stable          # 有输出 = rpm 版
dpkg -l google-chrome-stable         # 有输出 = deb 版
flatpak list | grep com.google.Chrome # 有输出 = flatpak 版
```

看装在哪、用的哪个源：

```bash
rpm -ql google-chrome-stable | grep -E 'chrome.desktop|widevine' | head
cat /usr/share/dnf5/repos.d/google-chrome.repo   # rpm 版的话，看 baseurl 指向哪
```

判断不出来就问用户，不要猜，更不要擅自换方式重装。

## 更新

**沿用检测出来的那种方式更新。**

rpm 版（dnf）：

```bash
sudo dnf update google-chrome-stable
```

deb 版（apt）：

```bash
sudo apt update && sudo apt install --only-upgrade google-chrome-stable
```

flatpak 版：

```bash
flatpak update com.google.Chrome
```

## 国内网络：官方源直连可用，阿里云镜像作为备用

**`dl.google.com` 和 `www.google.com` 在国内经常是整个被重置的**（2026-11 实测）：

```
$ curl -I https://dl.google.com/linux/chrome/deb/pool/main/g/google-chrome-stable/
curl: (35) Recv failure: 连接被对方重置
```

这样官方源那条路**有时候直连装不上**，得把 dnf 源指到国内镜像。

**但连通性会变**：2026-11-21 再测时 `dl.google.com` 已经能连上了
（`curl -I https://dl.google.com/linux/chrome/rpm/stable/x86_64/repodata/repomd.xml` 返回 200，
签名公钥 `https://dl.google.com/linux/linux_signing_key.pub` 也是 200），
同一天 `www.google.com` 的目录列表返回 404 而不是连接重置——说明当时的重置不是永久策略，
可能是按地区/时段生效，或者走了代理。
所以**每次装之前都先实测一次**：

```bash
curl -sI --max-time 15 https://dl.google.com/linux/chrome/rpm/stable/x86_64/repodata/repomd.xml | head -1
```

通了就直接用官方源（本机当前就是这个配置）；不通再走下面的阿里云镜像。

### 各镜像实测可用性（2026-11，官方源不通时的备用）

| 镜像 | 结果 |
| --- | --- |
| **阿里云** `https://mirrors.aliyun.com/google-chrome/google-chrome/` | ✓ 可用，rpm + `repodata/` 齐全，能直接当 dnf 仓库用 |
| 华为云 `https://repo.huaweicloud.com/google-chrome/` | ✗ 返回 401（要认证） |
| 清华 TUNA `https://mirrors.tuna.tsinghua.edu.cn/google-chrome/` | ✗ 404（已撤镜像） |
| 中科大 USTC `https://mirrors.ustc.edu.cn/google-chrome/` | ✗ 404 |
| 上海交大 SJTU `https://mirror.sjtu.edu.cn/google-chrome/` | ✗ 404 |
| 南京大学 `https://mirrors.nju.edu.cn/google-chrome/` | ✗ 404 |

**阿里云镜像的路径层级是 `google-chrome/google-chrome/`**（两层），
不是 `google-chrome/x86_64/` 也不是 `google-chrome/rpm/stable/x86_64/`——
按官方源的结构去猜路径会 404。

### 配镜像的完整步骤

**1. 导入 Google 签名公钥**（镜像上**没有**公钥文件，必须另找）：

```bash
curl -fL -o linux_signing_key.pub \
  "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0xfd533c07c264648f"
gpg --show-keys --with-fingerprint linux_signing_key.pub   # 核对指纹
sudo rpm --import linux_signing_key.pub
```

公钥指纹必须是 `EB4C 1BFD 4F04 2F6D DDCC EC91 7721 F63B D38B 4796`，
对应 key id `fd533c07c264648f`（Google Chrome 包的签名 key）。
指纹对不上就**别导**。

**2. 改仓库文件**（先备份）：

```bash
sudo cp /usr/share/dnf5/repos.d/google-chrome.repo{,.bak}
sudo tee /usr/share/dnf5/repos.d/google-chrome.repo >/dev/null <<'EOF'
[google-chrome]
name=google-chrome
baseurl=https://mirrors.aliyun.com/google-chrome/google-chrome/
skip_if_unavailable=True
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/GOOGLE-CHROME.gpg
enabled=1
EOF
```

也可以把公钥存成文件，`gpgkey=` 用 `file://` 指过去：

```bash
sudo cp linux_signing_key.pub /etc/pki/rpm-gpg/GOOGLE-CHROME.gpg
```

**3. 装**：

```bash
sudo dnf install google-chrome-stable
```

配好后 `sudo dnf update google-chrome-stable` 也会自动走镜像，不用再手动操作。

## 注意

- **Flathub 的 `com.google.Chrome` 在国内同样装不上**。
  它虽然挂在 Flathub 上，但**Chrome 二进制是从 `dl.google.com` 拉官方 deb 再转成 flatpak 的**，
  不是 Flathub 自己构建。所以 flatpak 装到最后一步会失败：

  ```
  错误： 安装 com.google.Chrome 失败：下载
  https://dl.google.com/linux/chrome/deb/pool/main/g/google-chrome-stable/
  google-chrome-stable_158.0.8127.36-1_amd64.deb 时：[35] SSL connect error
  ```

  前几步（运行时、平台、GL 扩展）会正常下完并留在本地，只有最后拉 Chrome 本体那步挂。
  flatpak 这条路**没有镜像可切**，别指望改源能救。
  （`dl.google.com` 连通性会变，2026-11-21 那次实测官方源已经能连了，但 flatpak 这条路没复测过。）

- **原装的 `google-chrome.repo` 里 `enabled=0`**（是之前装失败后被禁用的）。
  改镜像时记得同时把 `enabled` 改成 `1`，否则 `dnf install` 会说找不到包。

- **别用 `--nogpgcheck`**。Google 的包有签名，虽然过期 subkey 的警告很刺眼
  （`Subkey ... is expired`），但当前签名 key 是有效的，导入公钥后 `rpm -Kv` 会显示 `OK`。
  跳过校验等于放弃完整性验证。

- **Widevine 的路径不是 `libwidevinecdm.so`**。新版本 Chrome 把它放在
  `/opt/google/chrome/WidevineCdm/_platform_specific/linux_x64/libwidevinecdm.so`，
  用 `find /opt/google -iname '*widevine*'` 才找得到。Netflix 这类 DRM 站点能正常播放。

- **Chrome 依赖 SUID sandbox helper**：`/opt/google/chrome/chrome-sandbox` 需要是 `rwsr-xr-x`。
  正常 rpm 安装会自己设对；如果报
  `The SUID sandbox helper binary was found, but is not configured correctly`，
  执行 `sudo chmod 4755 /opt/google/chrome/chrome-sandbox`。

- **`google-chrome-stable_current_x86_64.rpm`** 是阿里云镜像上的「当前最新版」软链接，
  每次下到的版本会变；想固定版本就用 `google-chrome-stable-<版本>-1.x86_64.rpm` 那个具体文件。
  **注意它的路径在镜像根目录，不在 dnf 仓库目录里**：
  - ✓ `https://mirrors.aliyun.com/google-chrome/google-chrome-stable_current_x86_64.rpm`
  - ✗ `https://mirrors.aliyun.com/google-chrome/google-chrome/google-chrome-stable_current_x86_64.rpm`（404）

  仓库目录 `google-chrome/google-chrome/` 里**只有带版本号的 rpm**，没有 `current` 软链接。

## 本机状态（2026-11-21 装 / 同日复查）

- Fedora 45 (Workstation) / x86_64。
- **官方 rpm 装的是**，版本 **158.0.8127.36-1**（`rpm -q google-chrome-stable`），
  装在 `/opt/google/chrome/`，桌面项 `com.google.Chrome.desktop`。
- 仓库文件 `/usr/share/dnf5/repos.d/google-chrome.repo` **走官方源**，
  `baseurl=https://dl.google.com/linux/chrome/rpm/stable/x86_64`，`enabled=1`，
  `gpgkey=https://dl.google.com/linux/linux_signing_key.pub`（官方公钥，本机直连可下）。
  - 复查时发现这个文件一度是官方地址 + `enabled=0`（装失败后被禁用的状态），
    也就是**装好了但更新通路是断的**——`dnf update google-chrome-stable` 会说找不到包。
    改回 `enabled=1` 后 `dnf list --showduplicates google-chrome-stable` 正常列出全部版本。
  - 官方源不通时才换阿里云镜像（见上面「国内网络」一节），换法也记在那一节里。
- Google 公钥已导入 rpm keyring，副本在 `/etc/pki/rpm-gpg/GOOGLE-CHROME.gpg`。
- Widevine 在 `/opt/google/chrome/WidevineCdm/_platform_specific/linux_x64/`。
- SUID helper 正常：`/opt/google/chrome/chrome-sandbox` 是 `rwsr-xr-x`。
- 官方源上最新的 rpm 就是 `158.0.8127.36-1`，即已装版本，**没有可更新的**。
- **没装 flatpak 版**：flatpak 版本体要从 `dl.google.com` 拉官方 deb，
  `dl.google.com` 不通时这条路走不通（见上面「注意」第一条）。

### 复查时顺带发现的坑

- **`dnf check-update` 会被 Tailscale 源打断**：`pkgs.tailscale.com` 的仓库 GPG 签名验证失败
  （`repomd.xml GPG signature verification error: Signing key not found`），
  首次执行还会弹出交互式确认「Is this ok [y/N]」。这跟 Chrome 无关，是 Tailscale 源的 key 缺失，
  但会干扰 dnf 输出和非交互脚本——跑之前加 `dnf -y`。
  详见 `docs/mirrors.md`。


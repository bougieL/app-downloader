# 国内镜像源切换

把本机各种包管理器 / 软件源指向国内镜像（当前统一用 **中科大 USTC**：`https://mirrors.ustc.edu.cn`）。
记录已生效的配置、切换方法和踩过的坑，方便换机器或重装时照着做一遍。

## 本机当前生效的源

| 范围 | 工具 | 源地址 | 配置位置 |
| --- | --- | --- | --- |
| 系统 | dnf（Fedora / updates） | `https://mirrors.ustc.edu.cn/fedora/...` | `/etc/dnf/repos.override.d/00-ustc-mirror.repo` |
| 系统 | dnf（RPM Fusion nonfree） | `https://mirrors.ustc.edu.cn/rpmfusion/nonfree/...` | 同上 |
| 系统 | flatpak（flathub，system + user） | `https://mirrors.ustc.edu.cn/flathub` | `flatpak remote-modify` 写入 remote 配置 |
| 用户 | npm / pnpm | `https://registry.npmmirror.com/` | `~/.npmrc` |
| 用户 | pip | `https://mirrors.ustc.edu.cn/pypi/simple` | `~/.config/pip/pip.conf` |
| 用户 | cargo | `sparse+https://mirrors.ustc.edu.cn/crates.io-index/` | `~/.cargo/config.toml` |
| 用户 | go | `https://goproxy.cn` | `~/.config/go/env` |

没切的（见文末「保持原样的源」）：Chrome、Tailscale、Ghostty COPR、Fedora openh264、Fedora flatpak OCI 源。

Chrome 走的是官方源 `dl.google.com`（2026-11-21 实测直连可用），备用镜像记在 `docs/chrome.md`。

## dnf

### 怎么改的

Fedora 41 以后官方源定义搬到了 `/usr/share/dnf5/repos.d/`，这些文件归 `fedora-repos` 包管，
直接改会被系统更新覆盖。dnf5 的正规做法是写 override 文件 `/etc/dnf/repos.override.d/*.repo`。

本次用的是 `/etc/dnf/repos.override.d/00-ustc-mirror.repo`：

```ini
[fedora]
baseurl=https://mirrors.ustc.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
metalink=
mirrorlist=

[updates]
baseurl=https://mirrors.ustc.edu.cn/fedora/updates/$releasever/Everything/$basearch/
metalink=
mirrorlist=

[rpmfusion-nonfree-nvidia-driver]
baseurl=https://mirrors.ustc.edu.cn/rpmfusion/nonfree/fedora/nvidia-driver/$releasever/$basearch/
metalink=
mirrorlist=

[rpmfusion-nonfree-steam]
baseurl=https://mirrors.ustc.edu.cn/rpmfusion/nonfree/fedora/steam/$releasever/$basearch/
metalink=
mirrorlist=
```

改完刷新缓存：

```bash
sudo dnf makecache --refresh
```

### 坑一：metalink 优先于 baseurl，只写 baseurl 不生效

`dnf config-manager setopt fedora.baseurl=...` 写完之后 `dnf repoinfo` 里 Base URL 确实变了，
但 **dnf 实际下载时走的还是官方 metalink**——libdnf5 里 metalink 的优先级高于 baseurl，
metalink 存在就会被用上，baseurl 形同虚设。

验证办法：把 metalink 指到一个解析不了的域名，如果 dnf 报 `Curl error (6)` 而不是去拉 baseurl，
就说明 metalink 优先：

```bash
sudo tee /etc/yum.repos.d/_tmp_test.repo >/dev/null <<'EOF'
[_tmp_test]
name=tmp
metalink=https://invalid.example.invalid/repomd.xml
baseurl=https://mirrors.ustc.edu.cn/fedora/releases/45/Everything/x86_64/os/
gpgcheck=0
enabled=1
EOF
sudo dnf repoinfo --repoid=_tmp_test --refresh
sudo rm -f /etc/yum.repos.d/_tmp_test.repo
```

解决办法是在 override 里把 metalink **置空**（`metalink=`，空值，不是删掉那行——删掉等于没覆盖）。
置空后 `dnf repoinfo` 输出里 `Metalink` 一行会消失，只剩 `Base URL`，此时才真正走镜像。
`mirrorlist=` 同理，一起置空更保险。

### 坑二：config-manager 的 unsetopt 用不了

`dnf config-manager unsetopt fedora.metalink` 会报：

```
config-manager: Request to remove repository option but repoid is not present in the overrides
```

即使刚用 `setopt` 写进去过 baseurl、override 文件里明明有 `[fedora]` 这一节，也一样报。
所以「清掉 metalink」只能手写 override 文件，不要指望 unsetopt。

### 坑三：config-manager 会重写它自己的那个 override 文件

`/etc/dnf/repos.override.d/99-config_manager.repo` 是 `dnf config-manager` 生成的，
文件头写着「Do not modify this file manually」。它每次运行都会**整体重写**这个文件。
所以镜像配置要单独放一个自己的文件（本次是 `00-ustc-mirror.repo`），
别把 `metalink=` 手动塞进 `99-config_manager.repo`，否则下次跑 config-manager 就没了。

### 怎么确认真的走了镜像

```bash
sudo dnf repoinfo --repoid=fedora | grep -E 'Base URL|Metalink'   # 只剩 Base URL = 成功
sudo dnf makecache --refresh                                        # 不报 metalink 相关错误
dnf download --destdir=tmp/ --repoid=fedora --resolve hello          # 能下下来
```

## flatpak

flathub 的 system 和 user 两个 remote 都要改：

```bash
sudo flatpak remote-modify --system --url=https://mirrors.ustc.edu.cn/flathub flathub
flatpak remote-modify --user   --url=https://mirrors.ustc.edu.cn/flathub flathub
```

验证：

```bash
flatpak remotes --show-details --columns=name,options,url
flatpak remote-info --user flathub org.mozilla.firefox
```

USTC 的 flathub 是 yadex 布局（根目录有 `summary.idx`、`config`、`objects/`），
flatpak 的 OSTree 能直接读，不需要额外参数。

### 新建 user remote 要带 GPG key（2026-11 实测）

`flatpak remote-modify` **不支持 `--gpg-key`**，新建 remote 用的是 `remote-add`，
而且 flatpak 1.18 的选项名是 **`--gpg-import`**（不是老版本的 `--gpg-key`）：

```bash
flatpak remote-add --user --gpg-import=<keyring 文件> flathub https://mirrors.ustc.edu.cn/flathub
```

keyring 文件可以从 system remote 那里抄（system remote 是现成的）：

```bash
cp /var/lib/flatpak/repo/flathub.trustedkeys.gpg /tmp/flathub.gpg
flatpak remote-add --user --gpg-import=/tmp/flathub.gpg flathub https://mirrors.ustc.edu.cn/flathub
```

不导入 key 的症状是拉摘要时报签名错：

```
错误： 无法从远程仓库 flathub 加载摘要：Signature made ... using RSA key ID ...
Can't check signature: public key not found
```

**注意 `remote-add` 的参数顺序**：写成
`flatpak remote-add --user --gpg-import=<文件> flathub <URL>` 时，
实测 **URL 会被吃掉**（remote 被加成 `https://dl.flathub.org/repo/`）。
稳妥做法是分两步：先 `remote-add`（URL 用镜像），再 `remote-modify --user --url=...` 纠正。

## 用户级包管理器

```bash
# npm / pnpm
printf 'registry=https://registry.npmmirror.com/\n' >> ~/.npmrc
pnpm config get registry        # 应为 https://registry.npmmirror.com/

# pip
mkdir -p ~/.config/pip
cat > ~/.config/pip/pip.conf <<'EOF'
[global]
index-url = https://mirrors.ustc.edu.cn/pypi/simple
timeout = 30
EOF

# cargo
mkdir -p ~/.cargo
cat > ~/.cargo/config.toml <<'EOF'
[source.crates-io]
replace-with = "ustc"

[source.ustc]
registry = "sparse+https://mirrors.ustc.edu.cn/crates.io-index/"
EOF

# go
mkdir -p ~/.config/go
cat > ~/.config/go/env <<'EOF'
GOPROXY=https://goproxy.cn,direct
GOSUMDB=sum.golang.google.cn
EOF
```

## 保持原样的源（以及原因）

| 源 | 原因 |
| --- | --- |
| Chrome（`dl.google.com`） | 国内没有稳定镜像；官方源直连尚可。官方源不通时可换阿里云镜像，见 `docs/chrome.md` |
| Tailscale（`pkgs.tailscale.com`） | 同上。**但它的仓库 GPG key 没配好**，dnf 会报 `repomd.xml GPG signature verification error: Signing key not found`，首次还会弹交互确认——非交互跑 dnf 时加 `-y` |
| Ghostty COPR | 同上；COPR 本来就在 `copr.fedorainfracloud.org` |
| `fedora-cisco-openh264` | 仓库在 `codecs.fedoraproject.org/openh264/`，USTC / 清华 / 阿里都没有镜像；该源 `skip_if_unavailable=True`，拉不到不影响装包 |
| `fedora`（flatpak OCI 源） | `registry.fedoraproject.org`，没有通用国内镜像 |
| GitHub Release 下载 | 不走第三方加速站（ghproxy 之类不稳定），走本机的 Clash 本地代理，见下面「GitHub 下载」 |
| Homebrew | 本机没装 Linux 版 brew；且 USTC 只有 `brew.git` 和 bottles，**没有 homebrew-core**（`https://mirrors.ustc.edu.cn/homebrew-core.git` 404），要用 brew 得配清华的 `/git/homebrew/homebrew-core.git` |

## 各镜像可用性速查（2026-11 实测）

| 镜像 | 有 | 没有 |
| --- | --- | --- |
| USTC | fedora、updates、rpmfusion、flathub、pypi、crates.io-index、docker-ce、homebrew-bottles | cisco-openh264、debuginfo、homebrew-core |
| 清华 TUNA | fedora、updates、rpmfusion、pypi、crates、homebrew-bottles、homebrew git | flathub（已下架）、cisco-openh264 |
| 阿里云 | fedora、updates、pypi、crates | flathub（`os/x86_64` 布局 404）、cisco-openh264 |

需要 flathub 就只能 USTC；需要 homebrew-core 就只能清华。

## GitHub 下载

本机开着 Clash Verge（fake-ip + TUN 模式），**直连 GitHub 会失败**：

```
$ curl -sI https://github.com
curl: (35) TLS connect error: error:0A000126:SSL routines::unexpected eof while reading
```

DNS 解析出来的 `198.18.x.x` / `2001:2::` 是 fake-ip 地址，
流量被 TUN 接走后再按 Clash 的规则出站——规则给的是一条当前不通的节点，就表现为 TLS 握手中断。

**走本地代理端口就正常**（Clash 的 mixed 端口）：

```bash
export https_proxy=http://127.0.0.1:<代理端口> http_proxy=http://127.0.0.1:<代理端口>
curl -fsSL -o mise.tar.gz https://github.com/jdx/mise/releases/download/<版本>/mise-<版本>-linux-x64.tar.gz
```

### git 也需要显式配代理

**git 不读 `http_proxy` 环境变量**，只认 `http.proxy` 配置。
不配的话 `git clone` 会报上面那个 TLS 错：

```bash
git config --global http.proxy  http://127.0.0.1:<代理端口>
git config --global https.proxy http://127.0.0.1:<代理端口>
```

### 节点不稳时要重试

即使走了代理，节点也可能时通时断（同一个 URL 连续几次失败、下一次就成功）。
下载类操作统一写成带重试的循环，别把一次失败当成「没有这个资源」：

```bash
for i in 1 2 3 4 5 6; do
  timeout 300 git clone --depth=1 <url> <dir> && break
  rm -rf <dir>; sleep 4
done
```

大文件下载用 `curl -C -` 断点续传，重试时不会从头下。

## 回滚

```bash
sudo rm /etc/dnf/repos.override.d/00-ustc-mirror.repo
sudo dnf makecache --refresh
sudo flatpak remote-modify --system --url=https://dl.flathub.org/repo/ flathub
flatpak remote-modify --user   --url=https://dl.flathub.org/repo/ flathub
rm ~/.npmrc ~/.config/pip/pip.conf ~/.cargo/config.toml ~/.config/go/env
git config --global --unset http.proxy
git config --global --unset https.proxy
```

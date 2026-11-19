# tailscale

- 命令：`tailscale`（客户端）、`tailscaled`（守护进程，systemd 服务名 `tailscaled`）
- 仓库地址：https://github.com/tailscale/tailscale
- 官网：https://tailscale.com
- 官方安装脚本：https://tailscale.com/install.sh
- 官方 rpm 仓库：`https://pkgs.tailscale.com/stable/fedora/tailscale.repo`

基于 WireGuard 的组网工具（tailnet）。客户端二进制 + 需要 root 权限的 `tailscaled`
守护进程一起装，守护进程负责 TUN 设备、密钥交换和与控制服务器的通信。

## 重要：Tailscale 不走 GitHub Release

`tailscale/tailscale` 的 GitHub Release **不挂二进制资产**（`gh api .../releases/latest`
的 `assets` 为空），官方二进制只在自己的分发站：

- rpm / deb 仓库：`https://pkgs.tailscale.com/stable/<发行版>/`
- 静态用户态二进制（`tailscaled --tun=userspace-networking`）：官方安装脚本里的
  `tailscale_*.tgz` 包，需要自己从 `https://pkgs.tailscale.com/stable/tailscale_<版本>_linux_amd64.tgz` 取。

## 安装方式清单

装的时候选了哪种，以后就更新哪种。

| 安装方式 | 识别特征 | 沿用该方式的更新命令 |
| --- | --- | --- |
| 官方 rpm 仓库（`dnf install tailscale`，Fedora 默认选这个） | `rpm -q tailscale` 有输出，且 `/etc/yum.repos.d/tailscale.repo` 存在 | `sudo dnf upgrade tailscale` |
| 官方安装脚本 `curl -fsSL https://tailscale.com/install.sh \| sh` | 同上（脚本内部就是加 rpm 仓库再 `dnf install`） | 同上：`sudo dnf upgrade tailscale` |
| 手动下载 `.rpm` 用 `dnf install ./tailscale_*.rpm` | `rpm -q tailscale` 有输出，但仓库文件不存在 | 手动下载新版本 rpm 再 `sudo dnf install ./tailscale_*.rpm` |
| 静态二进制（userspace 模式，无 root） | 路径里有自己放的 `tailscaled` 二进制，且 `systemctl is-active tailscaled` 为 inactive | 手动替换二进制 |
| flatpak | `flatpak list \| grep -i tailscale` | `flatpak update` |

注意：Flathub 上**没有**官方 Tailscale 包，只有第三方的 GUI 客户端（Trayscale、KTailctl），
它们只能管理已安装的 `tailscaled`，不能替代安装。Tailscale 需要操作 TUN 设备，
flatpak 沙箱里做不了。

## 检测当前安装方式

```bash
rpm -q tailscale tailscaled        # 装没装
systemctl is-active tailscaled      # 服务在不在跑
readlink -f "$(command -v tailscale)" # 二进制在哪
ls /etc/yum.repos.d/tailscale.repo   # 是不是走官方仓库
```

## 更新

**沿用检测出来的安装方式，不要换方式。**

官方 rpm 仓库装的：

```bash
sudo dnf upgrade tailscale
```

仓库本身由 `tailscale.repo` 里的 `baseurl` 指向，不需要额外更新。

手动装的 rpm：去 `https://pkgs.tailscale.com/stable/fedora/x86_64/` 下载新版本 rpm，
`sudo dnf install ./tailscale_*.rpm` 覆盖升级。

## 认证

```bash
sudo tailscale up          # 打印登录链接，浏览器打开完成认证
tailscale status           # 看连接状态
tailscale ip -4            # 本机 tailnet IP
sudo tailscale logout      # 登出
```

`tailscale up` 会打印一条一次性登录链接（形如 `https://login.tailscale.com/a/...`），
需要人在浏览器里点开完成认证。**这条链接是一次性凭据，不要写进文档或提交进仓库。**

## zsh 支持

tailscale 自带 zsh 补全：

```bash
tailscale completion zsh > ~/.local/share/zsh/site-functions/_tailscale
```

本机把 `~/.local/share/zsh/site-functions` 加进了 `fpath`，补全文件由
`~/.oh-my-zsh/custom/conf/project-cli.zsh` 自动生成和维护，不用手动跑。
详见 `docs/oh-my-zsh.md`。

## 注意

### 走代理的网络环境里，tailscaled 要显式指定 HTTP 代理

**现象**：`tailscale up` 一直卡住不返回登录链接，journal 里偶发：

```
control: TryLogin: fetch control key: Get "https://controlplane.tailscale.com/key?v=148": context canceled
logtail: dial "log.tailscale.com:443" failed: dial tcp 198.18.0.37:443: i/o timeout (in 30s)
```

**原因**：本机的 DNS 被 clash / mihomo 的 fake-ip 接管（`198.18.0.0/16`），
`tailscaled` 按 fake-ip 直连时，有的域名映射是坏的（TLS 直接回
`tlsv1 alert internal error`），控制连接和日志上传就卡住。
`curl` 能通是因为它走了另一条 fake-ip 记录，不代表 `tailscaled` 也能通。

**解决办法**：让 `tailscaled` 显式走 mihomo 的 HTTP 代理端口，
用 systemd drop-in 注入环境变量：

```bash
sudo mkdir -p /etc/systemd/system/tailscaled.service.d
sudo tee /etc/systemd/system/tailscaled.service.d/override.conf >/dev/null <<'EOF'
[Service]
Environment=HTTPS_PROXY=http://127.0.0.1:<mihomo 混合端口>
Environment=HTTP_PROXY=http://127.0.0.1:<mihomo 混合端口>
Environment=NO_PROXY=127.0.0.1,localhost,::1
EOF
sudo systemctl daemon-reload
sudo systemctl restart tailscaled
```

混合端口在 clash-verge 的运行时配置 `mixed-port` 字段里，默认 `7897`。
加完之后 `sudo tailscale up` 就能正常打印登录链接了。

注意：

- `NO_PROXY` 里**不要**写 `*.tailscale.com`，否则控制连接又绕开代理，卡回老问题。
- 代理只作用于控制连接（`tailscaled` 用 `httpx` 读 `HTTPS_PROXY`），
  peer 之间的 WireGuard 流量不走它。
- 改了 drop-in 之后必须 `systemctl daemon-reload` + `restart tailscaled` 才生效。
- 本机没有 clash / mihomo 这类代理时不需要这个 drop-in，
  tailscaled 直连控制服务器即可。

### `tailscale up` 必须用 sudo

Linux 上 `tailscale up` 要通过本地 API 跟 root 跑的 `tailscaled` 通信，
普通用户执行会直接报错。非交互环境（脚本、CI）里用：

```bash
sudo tailscale up
```

### 后台跑 `tailscale up` 拿登录链接

`tailscale up` 会一直等认证完成，不适合直接丢在后台。用 `setsid` 完全脱离当前
shell，否则一断连就被 `context canceled` 掉：

```bash
setsid sudo tailscale up </dev/null >/tmp/ts-up.log 2>&1 &
```

日志里的 `To authenticate, visit:` 后面就是登录链接。

### 装完要启用服务

rpm 装完不会自动开机启动，手动开：

```bash
sudo systemctl enable --now tailscaled
```

### 版本和仓库

`tailscale` 官方 rpm 仓库在 `pkgs.tailscale.com/stable/fedora/`，目录结构按架构分
（`x86_64` / `aarch64`）。仓库带 GPG 签名校验，第一次 `dnf install` 会自动导入
Tailscale 的仓库签名密钥。

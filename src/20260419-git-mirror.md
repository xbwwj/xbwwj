# Git 镜像与 insteadOf

互联网并不稳定，例如 25 年底 [Cloudfare unwrap 大断网](https://blog.cloudflare.com/zh-cn/18-november-2025-outage), 又或者是未来的地缘风险。

因此对于重要的源码仓库，有必要缓存到本地。

## 创建 Git 镜像

```sh
git clone --mirror $repo
```

会将复制到本地作为 bare 仓库。

也可以选择放到 NAS 上，配合 mDNS:

```sh
git clone nas.local:/repos/github/$repo
```

## 用 insteadOf 切换到本地镜像

由于 Git submodules 的局限，日常使用镜像不够方便。最好是日常照常使用 GitHub, 仅在紧急情况换用本地镜像：

```sh
git config --global url."nas.local:/repos/github/".insteadOf "https://github.com/"
```

## 镜像自动更新

手动更新比较麻烦，可以配置服务器定期自动从上游镜像更新。

单个镜像的同步方式：

```sh
git fetch --all --prune
```

同步子目录所有镜像：

```sh
# sync-mirrors.sh
find /repos -name ".git" -type d -prune -execdir git fetch --all --prune \;
```

配置为系统服务：

```ini
# ~/.config/systemd/user/sync-mirrors.service
[Unit]
Description=Sync Mirrors
Wants=network-online.target
After=network-online.target

[Service]
Type=oneshot
WorkingDirectory=/repos/
ExecStart=sync-mirrors.sh;

[Install]
WantedBy=default.target
```

定时器：

```ini
# ~/.config/systemd/user/sync-mirrors.timer
[Unit]
Description=Sync mirrors every 2 hour

[Timer]
OnBootSec=5min
OnUnitActiveSec=2h
Persistent=true

[Install]
WantedBy=timers.target
```

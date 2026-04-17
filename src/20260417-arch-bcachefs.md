# Arch Linux 安装 bcachefs (dkms)

2026 年硬件价格高企，SSD 和 HDD 混合存储是最实惠的选择。

在传统的文件系统中，混合存储只能手动 `mount` 或者 `ln -s`,
或者选择拆分目录，比较麻烦。

## 解决方案：bcachefs

Linux 最新的 `bcachefs` 支持 SSD 和 HDD 混合存储，可以完美解决该问题。

可惜 6.18 之后的内核不再包含 `bcachefs`, 需要自己安装 dkms 模块。

本文记录我在 Arch Linux 上安装 `bcachefs-fkms` 的过程，以补网上资料阙漏。

> [!WARNING]
>
> 1. bcachefs 不够稳定，使用时务必注意资料备份。
> 2. 我只记录与官方 [installation](https://wiki.archlinux.org/title/Installation_guide) 有差异的章节（括号内）
>    - 请确保自己足够熟悉 Arch Linux, 否则请使用 `archinstall` 简易安装更简单的文件系统

## 1. 自建 archiso Live Install 镜像 (§1.1)

archiso 官方镜像中默认不包含 `bcachefs` 支持，无法挂载，需要自己创建。

```sh
pacman -S archiso
cp -r /usr/share/archiso/configs/releng/ archlive
cd archlive
```

在 `packages.x86_64` 中添加：

```
bcachefs-tools
bcachefs-dkms
linux-header
```

构建镜像并写入硬盘：

```sh
mkarchiso -v .
dd if=out/archlinux-xxxx.iso of=/dev/sdX bs=1M status=progress
sync
```

> [!NOTE]
>
> 如果你没有 Arch Linux 环境，或者已经删除了系统，可以在官方 archiso 中扩容
>
> ```sh
> mount -o remount,size=8G /run/archiso/cowspace
> ```
>
> 然后按照上述构建镜像。

## 2. 硬盘分区和格式化 (§1.9)

我有两块硬盘 `/dev/nvme0n1` 和 `/dev/sda`:

使用 `fdisk` 创建如下分区：

| 分区             | 大小 | 说明     |
| ---------------- | ---- | -------- |
| `/dev/nvme0n1p1` | 1G   | boot     |
| `/dev/nvme0n1p2` | *    | bcachefs |
| `/dev/sda1`      | *    | bcachefs |

```sh
mkfs.fat -F 32 /dev/nvme0n1p1

bcachefs format \
  --label=ssd.ssd1 /dev/nvme0n1p2 \
  --label=hdd.hdd1 /dev/sda1 \
  --replicas=1 \
  --foreground_target=ssd \
  --promote_target=ssd \
  --background_target=hdd \
  --compression=zstd
```

> [!WARNING]
> 
> - 我已经对重要资料另作备份，这里 `replicas=1` 无碍。
> - 自己按照需求创建 swap 分区

## 3. pacstrap (§2.2)

```sh
pacstrap -K /mnt base linux-zen linux-firmware \
  linux-zen-headers \
  bcachefs-tools bcachefs-dkms \
  neovim networkmanager
```

1. 这里我使用 `linux-zen`, 可以改用自己喜欢的内核
2. `linux-headers` 必须安装，否则 dkms 不会构建
3. `neovim` `networkmanager` 用于基本配置

## 4. 修改 initramfs (§3.6)

由于 bcachefs 目前是 dkms, 需要修改 initramfs.

编辑 `/etc/mkinitcpio.conf`, 在 `HOOKS` 中添加 `bcachefs`.

```conf
HOOKS=(... bcachefs filesystems ...)
```

生成对应 initramfs.

```sh
mkinitcpio -P
```

## 5. bootloader (§3.8)

这里我使用 systemd-boot.

```conf
# /boot/loader/entries/arch.conf 
title Arch Linux
linux /vmlinuz-linux-zen
initrd /initramfs-linux-zen.img
options root=UUID=<uuid> rw rootfstype=bcachefs
```

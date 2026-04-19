# 未曾设想的道路？在 Mesa/NVK 中直接支持 NVIDIA provided 驱动

> [!NOTE]
>
> NVIDIA 驱动版本较多，在内核驱动开发中通常如此称呼区分：
>
> - `nvidia`: NVIDIA proprietary
> - `nvidia-open`: NVIDIA provided
> - `nouveau`/`NOVA`: open source
>
> 由于 NOVA 并不准备支持 GSP. Turing 之前的显卡，不适用于本文情况。

## 历史

Linux 上的 N 卡驱动一直处于开闭源不兼容的状态。

传统上，开源驱动在内核空间使用 nouveau, 用户空间使用 mesa. mesa 仅支持 nouveau,
不支持 nvidia 或者 nvidia-open 驱动。

较新的的 Rust 驱动则使用内核 NOVA + 用户空间 NVK. 但是 NVK 依然仅支持开源驱动
NOVA, 不支持 nvidia-open.


## 转机

2025 年，Russel Greene 在 mesa 中提交了两个 PR：

- [Draft: NVK/nvkmd: Add support for nvidia.ko](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/34260)
- [Draft: NVK/nvkmd: Add support for nvgpu.ko](https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/34914)

旨在在 NVK 中直接强行兼容 NVIDIA-provided 驱动。其中 nvidia.ko
对应服务器和消费级主机显卡，nvgpu.ko 对应 Jetson 边缘设备。我们主要关注前者。

尽管还是草案状态，其已经初步支持了 Turing, Ampere, Backwell 架构显卡，并且能初步运行 vkcube 等程序。

但是稳定性还不足，dmabuf 等支持还没有实现。

## Waydroid 受益

Waydroid 通过 mesa 支持主机显卡。

由于此前 NVIDIA 使用 DRI 的方式与 mesa 不兼容，导致 Waydroid 安卓驱动无法利用 N
卡（[#1883](https://github.com/waydroid/waydroid/issues/1883)）。

一旦 NVK 对于 nvidia.ko 的支持成熟，就可以重新打包 mesa 层，实现 N 卡上运行 Waydroid.

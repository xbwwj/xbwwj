# 为什么我不用 NixOS

使用 NixOS 两年半后，我认为 NixOS 不能满足我的要求。

## 主观评价 NixOS

每种 Linux 发行版都有各自的优缺点。

NixOS 最吸引我的**优点**是：

1. **Flake**: 相比 AUR 而言，支持分散式包管理
2. **包生态**丰富之最：特别是 ARM 设备上

代价是 NixOS 日常使用中存在诸多**问题**：

1. **部署拖沓**：nixosConfiguration 已然，home-manager 更甚
2. **CUDA/Python** 库的安装非常折磨，即使有所谓 nixos-cuda mirror
3. **No impure**: Nix flake 不支持
   [lazy tree](https://github.com/NixOS/nix/pull/13225), 无法使用 monorepo
4. **空间占用大**：硬件涨价，磁盘空间越来越金贵
5. **内存需求高**：低内存边缘设备难以部署 nix-daemon (500MB 都不够?!)
6. **过度抽象**：屏蔽底层细节，没有 escape hatch, 难以修改

## 外部风险

以上缺陷只是**疥癣之疾**，真正的致命问题来自地缘政治风险：

1. 地缘摩擦加剧，局部冲突持续扩大
2. 各国经济下行，全球供应链面临挑战
3. 全世界右翼思潮抬头

**可以预见**在网络软件方面：

1. 国家间网络管控加强：清华等镜像可能不可用
2. 网络、电力等基础设施遭到破坏
3. **最极端情况**：离线、低功耗 ARM 设备，个人无法得到稳定的主机网络电源
   - 其实是手机
   - 乃至电子管？有点极端了

NixOS 则**极度依赖网络和算力**：

1. **浪费算力**：应用版本间强锁定，依赖项修改需要重新编译，边缘设备无从编译
2. **难以镜像**：NixOS = ~1TB, ArchLinux = ~100GB, 且 nix-store 复制麻烦
3. **难以修改**：PKGBUILD 可以轻松改用本地源码，Nix 修改 fetch 会导致整个缓存无效

## 替代品：brioche

我对于 Nix 最喜爱的功能是使用 Git 分散式打包，其实可以使用替代品 [brioche](https://github.com/brioche-dev/brioche):

1. 基于 JS (deno). 合理的模块系统 => 高性能低内存
2. 支持非 root 构建，无需 daemon

brioche 目前还是早期，且存在设计问题，可以未来 fork 或者 rewrite：

1. 包生态不完善
2. 缺少多版本并存（类 Arch）
3. nixpkgs 的模式优于其 std
4. 使用 quickjs 其实更好，构建系统不是瓶颈，省资源更重要。

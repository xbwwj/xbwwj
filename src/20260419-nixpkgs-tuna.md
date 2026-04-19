# 不止 binary cache, nixpkgs.git 也可以使用清华源

清华 TUNA 镜像本很提供了 Nix Channels [镜像](https://mirrors.tuna.tsinghua.edu.cn/help/nix-channels/), 提供了 NixOS 的 binary cache。

## 问题

但如果使用 Nix flake, 会需要从 GitHub 拉取 nixpkgs，依然需要特殊的网络才能运行。

## 解决

实际上 TUNA 除了同步 binary, 也同步上游源码。

可以通过以下方式配置 nix flake 也使用清华源：

```nix
inputs.nixpkgs.url = "git+https://mirrors.tuna.tsinghua.edu.cn/git/nixpkgs.git";
```

这样可以在受限网络中使用 Nix flake.

当然 nixos-cuda 等镜像的问题依然无法解决

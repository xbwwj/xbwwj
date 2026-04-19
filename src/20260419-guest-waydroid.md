# 虚拟机内 Waydroid 转发 adb

Waydroid 是 Linux 下最佳的 Android “模拟器”。但是其图形加速依赖安卓 mesa 驱动，导致 N 卡用户完全不可用。

目前主流的解决方案是先使用 QEMU 将 host GPU 抽象为 virtio-gpu，内层 guest 再使用 Waydroid 或者 redroid 运行安卓设备。虽然会有较大性能损失，但是至少可用，且对于安卓程序够用。

但是两层桥接导致 guest waydroid 无法直接被 host adb 连接。需要在 guest 中额外设置转发规则。

## 环境变量

- `LXC_ADDR=192.168.240.112` 容器地址
- `INTERFACE_ADDR=192.168.240.1` waydroid0 网卡地址

## 配置 guest 转发

```bash
# 1. 配置外部和主机访问 5555 端口转发到 LXC
sudo iptables -t nat -A PREROUTING -p tcp --dport 5555 -j DNAT --to-destination ${LXC_ADDR}:5555
sudo iptables -t nat -A OUTPUT -p tcp --dport 5555 -j DNAT --to-destination ${LXC_ADDR}:5555

# 2. 允许出入转发流量
sudo iptables -I FORWARD -d ${LXC_ADDR} -p tcp --dport 5555 -j ACCEPT
sudo iptables -I FORWARD -s ${LXC_ADDR} -p tcp --sport 5555 -j ACCEPT

## 3. 回程流量
sudo iptables -t nat -A POSTROUTING -s ${LXC_ADDR} -p tcp --sport 5555 -j SNAT --to-source ${INTERFACE_ADDR}
```

## 配置 host forward

QEMU 命令中需要额外参数：

```bash
  -net user,hostfwd=tcp::5555-:5555
```


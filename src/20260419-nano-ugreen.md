# Jetson Nano 安装绿联网卡驱动

绿联网卡使用 Realtek RTL8821CU 型号。对于 linux 6.12 以上版本的内核，内置 rtw88_8821cu 驱动。

但是 Jetson Nano 原生系统版本较老，需要自己编译驱动。

```sh
git clone https://github.com/brektrou/rtl8821CU
sudo ./dkms-install.sh

sudo usb_modeswitch -KW -v 0bda -p 1a2b
```

使用 udev 配置持久化：

```txt
# /lib/udev/rules.d/40-usb_modeswitch.rules
ATTR{idVendor}=="0bda", ATTR{idProduct}=="1a2b", RUN+="/usr/sbin/usb_modeswitch -K -v 0bda -p 1a2b"
```

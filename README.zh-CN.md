# AIC8800 D80 Linux 驱动

AIC8800 D80 USB Wi-Fi 驱动，厂商版本 6.4.3.0。

[English](README.md)

## 适用范围

适用于 AIC8800 D80 USB Wi-Fi 模块，包括 USB ID 为 `a69c:8d80`（AIC）与
`2357:014b`（TP-Link AIC 8800D80）的设备。

已验证平台：

- NVIDIA Jetson，内核 5.10.104-tegra，aarch64
- Ubuntu 22.04，内核 6.8.0-138-generic，x86_64

## 本版本修改

- 命令超时不再将命令管理器置为永久的崩溃状态；超时命令从队列移除并释放。
  连续 3 次超时会复位 USB 设备。
- 扫描、连接、断开失败时回滚驱动状态。
- 扫描增加 15 秒看门狗，超时通过 `cfg80211_scan_done()` 中止。
- USB 总线处于 down 状态时，`rwnx_send_msg()` 返回 `-ENODEV`。

## 编译

```bash
cd drivers/aic8800
make
sudo make install
```

## 固件

```bash
sudo ./install_setup.sh
```

脚本将 `fw/aic8800D80` 安装到 `/lib/firmware/`，并安装 udev 规则
`tools/aic.rules`。

# AIC8800 D80 Linux Driver

AIC8800 D80 USB Wi-Fi driver, vendor version 6.4.3.0.

[中文说明](README.zh-CN.md)

## Scope

AIC8800 D80 USB Wi-Fi modules, including devices with USB ID `a69c:8d80`
(AIC) and `2357:014b` (TP-Link AIC 8800D80).

Verified on:

- NVIDIA Jetson, kernel 5.10.104-tegra, aarch64
- Ubuntu 22.04, kernel 6.8.0-138-generic, x86_64

## Changes in this version

- A command timeout no longer leaves the command manager in a permanent
  crashed state; the timed-out command is removed from the queue and released.
  Three consecutive timeouts reset the USB device.
- Scan, connect and disconnect failures roll back driver state.
- A 15 s scan watchdog aborts a stuck scan through `cfg80211_scan_done()`.
- `rwnx_send_msg()` returns `-ENODEV` when the USB bus is down.

## Build

```bash
cd drivers/aic8800
make
sudo make install
```

## Firmware

```bash
sudo ./install_setup.sh
```

The script installs `fw/aic8800D80` to `/lib/firmware/` and the udev rule
`tools/aic.rules`.

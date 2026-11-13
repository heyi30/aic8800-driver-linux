# AIC8800 D80 Linux Driver

Linux USB Wi-Fi driver for the AIC8800 D80 chipset, vendor version 6.4.3.0.
Imported from `aic8800d80fdrvpackage.deb` and patched for robustness.

## Layout

| Path | Description |
|---|---|
| `drivers/aic8800/aic_load_fw/` | USB firmware loader |
| `drivers/aic8800/aic8800_fdrv/` | cfg80211 driver |
| `fw/aic8800D80/` | firmware and calibration files |
| `tools/aic.rules` | udev rule |
| `install_setup.sh` | installs firmware and udev rule |
| `uninstall_setup.sh` | removes firmware and udev rule |

## Local patches

Applied 2026-11-13 on an NVIDIA Jetson (kernel 5.10.104-tegra, aarch64):

- `rwnx_cmds.c`: a command timeout no longer moves the command manager into a
  permanent `RWNX_CMD_MGR_STATE_CRASHED` state. The timed-out command is
  removed from the queue and released. Three consecutive timeouts schedule a
  USB device reset.
- `aicwf_usb.c`: adds a reset work item that calls `usb_queue_reset_device()`,
  forcing re-enumeration and firmware reload.
- `rwnx_msg_tx.c`: scan request failures clear `scanning` and `scan_request`;
  `rwnx_send_msg()` returns `-ENODEV` when the USB bus is down.
- `rwnx_main.c`: scan, connect and disconnect failure paths roll back driver
  state; a 15 s scan watchdog aborts a stuck scan through
  `cfg80211_scan_done()`; removed `WARN_ON()` from recoverable state-machine
  paths.
- `rwnx_msg_rx.c`: cancels the scan watchdog on `SCAN_DONE_IND`.

## Build

Tested with kernel 5.10.104-tegra:

```bash
cd drivers/aic8800
make
sudo make install
```

`make install` copies the modules to
`/lib/modules/$(uname -r)/kernel/drivers/net/wireless/aic8800/` and runs
`depmod`.

## Firmware

```bash
sudo ./install_setup.sh
```

The script copies `fw/aic8800D80` to `/lib/firmware/`, installs
`tools/aic.rules` and triggers udev.

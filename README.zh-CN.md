# AIC8800 D80 Linux 驱动

AIC8800 D80 芯片的 Linux USB Wi-Fi 驱动，厂商版本 6.4.3.0。源码取自
`aic8800d80fdrvpackage.deb`，并应用了健壮性补丁。

[English](README.md)

## 目录结构

| 路径 | 说明 |
|---|---|
| `drivers/aic8800/aic_load_fw/` | USB 固件加载器 |
| `drivers/aic8800/aic8800_fdrv/` | cfg80211 驱动 |
| `fw/aic8800D80/` | 固件与校准文件 |
| `tools/aic.rules` | udev 规则 |
| `install_setup.sh` | 安装固件与 udev 规则 |
| `uninstall_setup.sh` | 卸载固件与 udev 规则 |

## 本地补丁

2026-11-13 在 NVIDIA Jetson（内核 5.10.104-tegra，aarch64）上应用：

- `rwnx_cmds.c`：命令超时不再将命令管理器置为永久的
  `RWNX_CMD_MGR_STATE_CRASHED` 状态。超时命令从队列移除并释放。连续 3 次
  超时调度 USB 设备复位。
- `aicwf_usb.c`：新增复位工作项，调用 `usb_queue_reset_device()`，触发设备
  重新枚举与固件重新加载。
- `rwnx_msg_tx.c`：扫描请求发送失败时清除 `scanning` 与 `scan_request`；
  USB 总线处于 down 状态时 `rwnx_send_msg()` 返回 `-ENODEV`。
- `rwnx_main.c`：扫描、连接、断开失败路径回滚驱动状态；15 秒扫描看门狗
  通过 `cfg80211_scan_done()` 中止卡住的扫描；移除可恢复状态机路径中的
  `WARN_ON()`。
- `rwnx_msg_rx.c`：收到 `SCAN_DONE_IND` 时取消扫描看门狗。

## 编译

在 5.10.104-tegra 上测试：

```bash
cd drivers/aic8800
make
sudo make install
```

`make install` 将模块复制到
`/lib/modules/$(uname -r)/kernel/drivers/net/wireless/aic8800/` 并执行
`depmod`。

## 固件

```bash
sudo ./install_setup.sh
```

脚本将 `fw/aic8800D80` 复制到 `/lib/firmware/`，安装 `tools/aic.rules`
并触发 udev。

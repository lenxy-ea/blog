---
title: 拯救 BlueField-2
date: 2026-04-20 17:26:58
tags: 灵车日记
---

# 拯救 BlueField-2

> 呜呜呜，我那么大块网卡怎么说暴毙就暴毙啊

今天开机的时候发现机器莫名其妙的慢慢慢慢，于是看了一眼dmesg

发现我那么大的BlueField2 2没起来

```sh
[    2.818323] mlx5_core 0000:6f:00.0: PTM is not supported by PCIe
[    2.823757] mlx5_core 0000:6f:00.0: firmware version: 24.47.1026
[    2.826445] mlx5_core 0000:6f:00.0: 126.024 Gb/s available PCIe bandwidth (16.0 GT/s PCIe x8 link)
[   22.829549] mlx5_core 0000:6f:00.0: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 100s (0x87010000)
[   42.839517] mlx5_core 0000:6f:00.0: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 79s (0x87010000)
[   62.850549] mlx5_core 0000:6f:00.0: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 59s (0x87010000)
[   82.858549] mlx5_core 0000:6f:00.0: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 39s (0x87010000)
[  102.870547] mlx5_core 0000:6f:00.0: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 19s (0x87010000)
[  122.830548] mlx5_core 0000:6f:00.0: wait_fw_init:201:(pid 8): Firmware over 120000 MS in pre-initializing state, aborting
[  122.835105] mlx5_core 0000:6f:00.0: probe_one:1987:(pid 8): mlx5_init_one failed with error code -110
[  122.842813] mlx5_core: probe of 0000:6f:00.0 failed with error -110
[  122.849733] mlx5_core 0000:6f:00.1: PTM is not supported by PCIe
[  122.854385] mlx5_core 0000:6f:00.1: firmware version: 24.47.1026
[  122.859021] mlx5_core 0000:6f:00.1: 126.024 Gb/s available PCIe bandwidth (16.0 GT/s PCIe x8 link)
[  142.867547] mlx5_core 0000:6f:00.1: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 100s (0x87010000)
[  162.878546] mlx5_core 0000:6f:00.1: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 79s (0x87010000)
[  182.891546] mlx5_core 0000:6f:00.1: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 59s (0x87010000)
[  202.905546] mlx5_core 0000:6f:00.1: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 39s (0x87010000)
[  222.919545] mlx5_core 0000:6f:00.1: wait_fw_init:211:(pid 8): Waiting for FW pre-initializing, timeout abort in 19s (0x87010000)
[  242.867545] mlx5_core 0000:6f:00.1: wait_fw_init:201:(pid 8): Firmware over 120000 MS in pre-initializing state, aborting
[  242.873377] mlx5_core 0000:6f:00.1: probe_one:1987:(pid 8): mlx5_init_one failed with error code -110
[  242.882154] mlx5_core: probe of 0000:6f:00.1 failed with error -110
```

## 可能的病因

主机服务器上的驱动程序依赖于 **Arm 侧**。如果 **Arm 侧** 的驱动正常启动，那么主机服务器上的驱动也会随之正常工作。

请确认以下几点：

- **BlueField（Arm 侧）** 的驱动程序已经正确加载
- **Arm 侧** 已经正常启动进入操作系统
- **Arm 侧** 当前没有停留在 **UEFI Boot Menu**
- **Arm 侧** 没有发生卡死或无响应的情况

## 解决方法

直接重刷bfd

```bash
mst start
setenforce 0 #在注意SELinux，不关闭会造成rshim报错无法启动。
systemctl start rshim
bfb-install --bfb bf-bundle-3.3.0-202_26.01_ubuntu-24.04_64k_prod.bfb --config bf.cfg --rshim rshim0
echo "SW_RESET 1" > /dev/rshim0/mis
```

简单粗暴重刷后就没有这个异常了。

参考文献



[R750 DSS NVIDIA Mellanox BlueField-2 DPU Card DPN-GRNMC PCIe link training failure](https://www.dell.com/support/kbdoc/en-us/000228342/r750-dss-mellanox-bluefield-2-dpu-card-dpn-grnmc-have-multiple-failure)

[BlueFiled Connectivity Troubleshooting](https://docs.nvidia.com/networking/display/bluefielddpuosv3931/connectivity+troubleshooting)


---
title: 探索Tplink 7VR7200L
date: 2026-08-03 15:43:11
tags: 灵车日记
---

# 探索Tplink 7VR7200L

最近帮朋友刷系统，摸到了一台Tplink的原型机！对应型号应该是7DR7260。

还是来先看一下芯片吧

CPU: MediaTek MT7988DV (A73, 3x 1.8GHz, HWNAT, 2.5 PHY) (Filogic 860)

Memory: GigaDevice GDQ2BFAA-CJ 512MB

Flash: ESMT F50L1G41LB 128MB

WLAN: 

MediaTek MT7992AV (2.4/5G), 128MB/512MB (Filogic 660)
MediaTek MT7975N (2.4G 4SS-40MHz-4K QAM)
MediaTek MT7979N (5G 4SS-160MHZ-4K QAM)

Switch: Realtek RTL8372N (2.5G)

LAN: RTL8221B(本板缺件，导致第二个口不可用)

##  插电开机测试

先把uart接好。bl不再是原版，而是商家给的imm修改版。估计想要自己日会比较麻烦了。

uboot里会看到个有趣的事，*SPI_NAND Detected ID 0xc8*  在Uboot里识别是GigaDevice SPI NAND was found而在openwrt里面则会显示ESMT。

这里就得提到一件趣事，Linux的spi-nand驱动注释里明确写着"ESMT喜欢用别家厂商的JEDEC ID"，1G芯片用GigaDevice的0xC8，2G/4G用Micron的0x2C。

所以不要被这里误导了！

```
F0: 102B 0000
FA: 1042 0000
FA: 1042 0000 [0200]
F9: 0000 0000
V0: 0000 0000 [0001]
00: 0000 0000
BP: 0600 0041 [0000]
G0: 1190 0000
EC: 0000 0000 [1000]
MK: 0000 0000 [0000]
T0: 0000 01A8 [0101]
Jump to BL

NOTICE:  BL2: v2.13.0(release):ImmortalWrt v2025.07.11~78a0dfd9-1 (mt7988-spim-nand-ddr4)
NOTICE:  BL2: Built : 08:37:24, Sep  2 2025
NOTICE:  WDT: Cold boot
NOTICE:  WDT: disabled
NOTICE:  CPU: MT7988
NOTICE:  EMI: Using DDR4 settings
NOTICE:  EMI: Detected DRAM size: 512 MB
NOTICE:  EMI: complex R/W mem test passed
NOTICE:  LVTS: Enable thermal HW reset
NOTICE:  SPI_NAND parses attributes from parameter page.
NOTICE:  SPI_NAND Detected ID 0xc8
NOTICE:  Page size 2048, Block size 131072, size 134217728
NOTICE:  BL2: Booting BL31
NOTICE:  BL31: v2.13.0(release):071e4498-dirty
NOTICE:  BL31: Built : 17:45:02, Aug 24 2025

U-Boot 2025.07 (Aug 24 2025 - 16:53:07 +0800)

CPU:   MediaTek MT7988
Model: TP-Link TL-7DR7260 v2
       (mediatek,mt7988-rfb)
DRAM:  512 MiB
Core:  34 devices, 12 uclasses, devicetree: separate

Initializing NMBM ...
spi-nand: spi_nand spi_nand@0: CASN page check failed
spi-nand: spi_nand spi_nand@0: Fallback to read ID
spi-nand: spi_nand spi_nand@0: GigaDevice SPI NAND was found.
spi-nand: spi_nand spi_nand@0: 128 MiB, block size: 128 KiB, page size: 2048, OOB size: 64
Could not find a valid device for nmbm0
Signature found at block 1023 [0x07fe0000]
First info table with writecount 0 found in block 960
Second info table with writecount 0 found in block 963
NMBM has been successfully attached

Loading Environment from MTD... OK
In:    serial@11000000
Out:   serial@11000000
Err:   serial@11000000
Net:   realtek rtl8372n
```


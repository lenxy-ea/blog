---
title: Upgarde a old N3K
date: 2026-05-08 11:09:10
tags: 日常
---

# 老旧Cisco N3K升级改造

简单记录下手里有一台老旧的Cisco Nexus设备下升级流程，升级需要注意要同时处理`kickstart` + `system` 。

kickstart 提供另一个最小的启动环境。完成与硬件的初始化，（还有点类似bootloader的东西）。kickstart完成后才会引导system image进入。所以这两个程序的版本都需要匹配，如果不匹配会造成启动失败。

```
启动引导图
Golden BIOS 
   ↓
kickstart image
   ↓
system image
   ↓
完整 NX-OS
```

升级前务必参考文档，确定你的中间过渡版本 [Cisco Nexus 3000 Upgrade ](https://www.cisco.com/c/en/us/support/docs/switches/nexus-3000-series-switches/216037-nexus-3000-and-3100-nx-os-software-upgra.html#toc-hId-330091406) 

剩下的内容就简单多了，只需要两条命令（如果不进行验证 一条命令也可以）



```shell
# 检测升级影响
# show install all impact kickstart bootflash:///n3000-uk9-kickstart.6.0.2.U6.10.bin system bootflash:///n3000-uk9.6.0.2.U6.10.bin 

Installer is forced disruptive

Verifying image bootflash:/n3000-uk9-kickstart.6.0.2.U6.10.bin for boot variable "kickstart".
[########################################] 100% -- SUCCESS

Verifying image bootflash:/n3000-uk9.6.0.2.U6.10.bin for boot variable "system".
[########################################] 100% -- SUCCESS

Verifying image type.
[########################################] 100% -- SUCCESS

Extracting "system" version from image bootflash:/n3000-uk9.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Extracting "kickstart" version from image bootflash:/n3000-uk9-kickstart.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Extracting "bios" version from image bootflash:/n3000-uk9.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Performing module support checks.
[########################################] 100% -- SUCCESS

Notifying services about system upgrade.
[########################################] 100% -- SUCCESS



Images will be upgraded according to following table:
Module             Image         Running-Version             New-Version  Upg-Required
------  ----------------  ----------------------  ----------------------  ------------
     1            system             6.0(2)U3(7)            6.0(2)U6(10)           yes
     1         kickstart             6.0(2)U3(7)            6.0(2)U6(10)           yes
     1              bios      v2.6.0(08/06/2014)      v2.8.0(12/22/2015)           yes
     1         power-seq                    v1.0                    v1.0            no
     1            SFP-uC                   v2.12                   v2.12            no

# 执行升级
# install all kickstart bootflash:///n3000-uk9-kickstart.6.0.2.U6.10.bin system bootflash:///n3000-uk9.6.0.2.U6.10.bin  
Installer is forced disruptive

Verifying image bootflash:/n3000-uk9-kickstart.6.0.2.U6.10.bin for boot variable "kickstart".
[########################################] 100% -- SUCCESS

Verifying image bootflash:/n3000-uk9.6.0.2.U6.10.bin for boot variable "system".
[########################################] 100% -- SUCCESS

Verifying image type.
[########################################] 100% -- SUCCESS

Extracting "system" version from image bootflash:/n3000-uk9.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Extracting "kickstart" version from image bootflash:/n3000-uk9-kickstart.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Extracting "bios" version from image bootflash:/n3000-uk9.6.0.2.U6.10.bin.
[########################################] 100% -- SUCCESS

Performing module support checks.
[########################################] 100% -- SUCCESS

Notifying services about system upgrade.
[########################################] 100% -- SUCCESS



Compatibility check is done:
Module  bootable          Impact  Install-type  Reason
------  --------  --------------  ------------  ------
     1       yes      disruptive         reset  Forced by the user



Images will be upgraded according to following table:
Module             Image         Running-Version             New-Version  Upg-Required
------  ----------------  ----------------------  ----------------------  ------------
     1            system             6.0(2)U3(7)            6.0(2)U6(10)           yes
     1         kickstart             6.0(2)U3(7)            6.0(2)U6(10)           yes
     1              bios      v2.6.0(08/06/2014)      v2.8.0(12/22/2015)           yes
     1         power-seq                    v1.0                    v1.0            no
     1            SFP-uC                   v2.12                   v2.12            no


Switch will be reloaded for disruptive upgrade.
Do you want to continue with the installation (y/n)?  [n] y
Time Stamp: Sat Feb  7 15:43:12 2026


Install is in progress, please wait.

Performing runtime checks.
[########################################] 100% -- SUCCESS

Setting boot variables.
[########################################] 100% -- SUCCESS

Performing configuration copy.
[########################################] 100% -- SUCCESS

Module 1: Refreshing compact flash and upgrading bios/loader/bootrom/power-seq.
Warning: please do not remove or power off the module at this time.
Note: Power-seq upgrade needs a power-cycle to take into effect.
On success of power-seq upgrade, SWITCH OFF THE POWER to the system and then, power it up.
[#                                       ]   0%

              
# show ver

Software
  BIOS:      version 2.8.0
  loader:    version N/A
  kickstart: version 6.0(2)U6(10)
  system:    version 6.0(2)U6(10)
  Power Sequencer Firmware:
             Module 1: version v1.0
  SFP uC:    version 2.12
  BIOS compile time:       12/22/2015
  kickstart image file is: bootflash:///n3000-uk9-kickstart.6.0.2.U6.10.bin
  kickstart compile time:  3/30/2017 9:00:00 [03/30/2017 16:37:34]
  system image file is:    bootflash:///n3000-uk9.6.0.2.U6.10.bin
  system compile time:     3/30/2017 9:00:00 [03/30/2017 17:04:06]



```

后续升级按照path即可，升级到7.x的版本不需要kickstart了

---
title: Oh My XikeStor SKS3200-8E1X
date: 2026-05-26 21:03:51
tags: 灵车日记
---

# 兮克 SKS3200-8E1X 暴毙日记

> 其实这篇文章应该更早出的，但是一直咕咕咕。

之前我的兮克跑老版本的固件虽然间歇性管理平面死机，DHCP包直接被阻拦，但是不影响我正常使用。直到有一天群友跟我说 **去升级固件啊**！

升级完固件交换机就进入了无限循环往复。

```
===========================Config Area pre-check Starts.=====================.
Pre-Check the config size structure is equal or not.
(sizeof(configCache)) a68.
(FLSH_ADDR_END-FLSH_CONFIG_ADDR_START) a68.
(FLSH_CONFIG_ADDR_START) 1fe000.
(FLSH_ADDR_END) 1fea68.
It seems no risk!..................
==============================Config Area pre-check ends.===================.



SalFlshCopyFlshToCache()
sal_sys_config_restore()
Restore dhcp state is: 0

Restore ip is: 172.16.10.17

...OK
sal_mirror_config_restore()...OK
sal_qos_config_restore()...OK
sal_vlan_config_restore()...OK
sal_rate_config_restore()...OK
sal_trunk_config_restore()...OK
sal_l2_config_restore()...OK
sal_loop_config_restore()...OK
sal_eee_config_restore()...OK
sal_stp_config_restore()...OK
sal_igmp_config_restore()...OK
sal_dhcp_snooping_config_restore()...OK
sal_port_config_restore()...OK


==========Loader start V0.2===========
Press any key to start the normal procedure.
To run SPI flash viewer, press [v]
To enforce the download of the runtime kernel, press [ESC] .....
  cmd -1
    Check Runtime Image.....
    Chksum Correct!
    RunTime Kernel Starting....
Ver8372N=2
Ver8373N=2
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf finished!
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf0 finished!



===========================Config Area pre-check Starts.=====================.
Pre-Check the config size structure is equal or not.
(sizeof(configCache)) a68.
(FLSH_ADDR_END-FLSH_CONFIG_ADDR_START) a68.
(FLSH_CONFIG_ADDR_START) 1fe000.
(FLSH_ADDR_END) 1fea68.
It seems no risk!..................
==============================Config Area pre-check ends.===================.



SalFlshCopyFlshToCache()
sal_sys_config_restore()
Restore dhcp state is: 0

Restore ip is: 172.16.10.17

...OK
sal_mirror_config_restore()...OK
sal_qos_config_restore()...OK
sal_vlan_config_restore()...OK
sal_rate_config_restore()...OK
sal_trunk_config_restore()...OK
sal_l2_config_restore()...OK
sal_loop_config_restore()...OK
sal_eee_config_restore()...OK
sal_stp_config_restore()...OK
sal_igmp_config_restore()...OK
sal_dhcp_snooping_config_restore()...OK
sal_port_config_restore()...OK


==========Loader start V0.2===========
Press any key to start the normal procedure.
To run SPI flash viewer, press [v]
To enforce the download of the runtime kernel, press [ESC] .....
  cmd -1
    Check Runtime Image.....
    Chksum Correct!
    RunTime Kernel Starting....
Ver8372N=2
Ver8373N=2
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf finished!
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf0 finished!



===========================Config Area pre-check Starts.=====================.
Pre-Check the config size structure is equal or not.
(sizeof(configCache)) a68.
(FLSH_ADDR_END-FLSH_CONFIG_ADDR_START) a68.
(FLSH_CONFIG_ADDR_START) 1fe000.
(FLSH_ADDR_END) 1fea68.
It seems no risk!..................
==============================Config Area pre-check ends.===================.



SalFlshCopyFlshToCache()
sal_sys_config_restore()
Restore dhcp state is: 0

Restore ip is: 172.16.10.17

...OK
sal_mirror_config_restore()...OK
sal_qos_config_restore()...OK
sal_vlan_config_restore()...OK
sal_rate_config_restore()...OK
sal_trunk_config_restore()...OK
sal_l2_config_restore()...OK
sal_loop_config_restore()...OK
sal_eee_config_restore()...OK
sal_stp_config_restore()...OK
sal_igmp_config_restore()...OK
sal_dhcp_snooping_config_restore()...OK
sal_port_config_restore()...OK


==========Loader start V0.2===========
Press any key to start the normal procedure.
To run SPI flash viewer, press [v]
To enforce the download of the runtime kernel, press [ESC] .....
  cmd -1
    Check Runtime Image.....
    Chksum Correct!
    RunTime Kernel Starting....
Ver8372N=2
Ver8373N=2
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf finished!
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf0 finished!



===========================Config Area pre-check Starts.=====================.
Pre-Check the config size structure is equal or not.
(sizeof(configCache)) a68.
(FLSH_ADDR_END-FLSH_CONFIG_ADDR_START) a68.
(FLSH_CONFIG_ADDR_START) 1fe000.
(FLSH_ADDR_END) 1fea68.
It seems no risk!..................
==============================Config Area pre-check ends.===================.



SalFlshCopyFlshToCache()
sal_sys_config_restore()
Restore dhcp state is: 0

Restore ip is: 172.16.10.17

...OK
sal_mirror_config_restore()...OK
sal_qos_config_restore()...OK
sal_vlan_config_restore()...OK
sal_rate_config_restore()...OK
sal_trunk_config_restore()...OK
sal_l2_config_restore()...OK
sal_loop_config_restore()...OK
sal_eee_config_restore()...OK
sal_stp_config_restore()...OK
sal_igmp_config_restore()...OK
sal_dhcp_snooping_config_restore()...OK
sal_port_config_restore()...OK


==========Loader start V0.2===========
Press any key to start the normal procedure.
To run SPI flash viewer, press [v]
To enforce the download of the runtime kernel, press [ESC] ...
  cmd 27
sal_sys_runtime_crc_set
loader start
Ver8372N=2
Ver8373N=2
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf finished!
LINE 1462, RL6818C_pwr_on_patch_phy_v007 , patch 0xf0 finished!
load MAC from nvcfg
  IP:192.168.1.1
Mask:255.255.255.0
  GW:192.168.1.254



```

没错，这个东西一直在重启，我的弱电箱核心交换机就此暴毙。我也不得不变成原始人

从此以后我痛定思痛下单了Mikrotik CRS310，

# 等我的编程器到了我将会继续大战 等我回来！！！！！

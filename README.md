# AidoSDRpatch
This Repository is used to make AidoSDR software radio device firmware. 



## Build Instructions

The Firmware is built with the [Xilinx Vivado 2023.2](https://account.amd.com/en/forms/downloads/xef.html?filename=FPGAs_AdaptiveSoCs_Unified_2023.2_1013_2256.tar.gz)(v0.39). You need to install the correct Vivado version in you Linux PC, and then,you can follow the instructions below to generate the firmware for 

### Install build requirements

```sh
sudo apt-get install git build-essential fakeroot libncurses5-dev libssl-dev ccache 
sudo apt-get install dfu-util u-boot-tools device-tree-compiler mtools
sudo apt-get install bc python cpio zip unzip rsync file wget 
sudo apt-get install libtinfo5 device-tree-compiler bison flex u-boot-tools
sudo apt-get purge gcc-arm-linux-gnueabihf
sudo apt-get remove libfdt-de
```

### Get source code and setup bash

1. get source from git
	- v0.39
		
		```sh
		git clone --recursive https://github.com/sdrdeepaido/aidosdr-fw.git
		```
2. Toolchain

   Due to incompatibility between the AMD/Xilinx GCC toolchain supplied with Vivado/Vitis and Buildroot. This project switched to Buildroot external Toolchain: Linaro GCC 7.3-2018.05 7.3.1
   https://releases.linaro.org/components/toolchain/binaries/7.3-2018.05/arm-linux-gnueabihf/


3. setup bash
	- v0.39
        ```sh
        export CROSS_COMPILE=arm-linux-gnueabihf-
        export PATH=$PATH:/Toolchain-PATH/gcc-linaro-7.3.1-2018.05-i686_arm-linux-gnueabihf/bin
        export VIVADO_SETTINGS=/opt/Xilinx/Vivado/2023.2/settings64.sh
        ```

### Export target
   ```sh
   export TARGET=AidoSDR
   ```

  

### Patch

After completing the above steps, start to Patch.

```sh
cd aidosdr-fw
```

   ```sh
   
   sh patch.sh AidoSDR
   ```

   

If you patch is successfully applied, you can see the following information.

```txt
AIdo_sdr/aidosdr-fw$ sh patch.sh AidoSDR
Patch check...
 ...
 ...

Patch...
...
...
patch finish

```

### Build

Then you can make firmware.

```sh
cd plutosdr-fw
sudo -E make
```

After the firmware building finished, you will see below file in the build folder. These files are used for flash updating.

```txt
AIdo_sdr/aidosdr-fw/plutosdr-fw$ ls -AGhl build
总计 572M
-rw-r--r-- 1 root  15M  6月 11 17:46 AidoSDR.dfu
-rw-r--r-- 1 root  15M  6月 11 17:47 AidoSDR.frm
-rw-r--r-- 1 root   33  6月 11 17:47 AidoSDR.frm.md5
-rw-r--r-- 1 root  15M  6月 11 17:46 AidoSDR.itb
-rw-r--r-- 1 root   69  6月 11 17:47 boot.bif
-rw-r--r-- 1 root 510K  6月 11 17:47 boot.bin
-rw-r--r-- 1 root 510K  6月 11 17:47 boot.dfu
-rw-r--r-- 1 root 639K  6月 11 17:47 boot.frm
-rw-r--r-- 1 root 480M  6月 11 17:47 legal-info-v0.39-dirty.tar.gz
-rw-r--r-- 1 root 641K  6月 11 17:36 LICENSE.html
-rw-r--r-- 1 root  26M  6月 11 17:47 plutosdr-fw-v0.39-dirty.zip
-rw-r--r-- 1 root 720K  6月 11 17:47 plutosdr-jtag-bootstrap-v0.39-dirty.zip
-rw-r--r-- 1 root 524K  6月 11 17:45 ps7_init.c
-rw-r--r-- 1 root 525K  6月 11 17:45 ps7_init_gpl.c
-rw-r--r-- 1 root 4.2K  6月 11 17:45 ps7_init_gpl.h
-rw-r--r-- 1 root 3.6K  6月 11 17:45 ps7_init.h
-rw-r--r-- 1 root 2.8M  6月 11 17:45 ps7_init.html
-rw-r--r-- 1 root  35K  6月 11 17:45 ps7_init.tcl
-rw-r--r-- 1 root 7.9M  6月 11 17:37 rootfs.cpio.gz
drwxr-xr-x 6 root 4.0K  6月 11 17:46 sdk
-rw-r--r-- 1 root 2.4M  6月 11 17:45 system_top.bit
-rw-r--r-- 1 root 858K  6月 11 17:45 system_top.xsa
-rwxr-xr-x 1 root 473K  6月 11 17:47 u-boot.elf
-rw-r----- 1 root 128K  6月 11 17:47 uboot-env.bin
-rw-r----- 1 root 129K  6月 11 17:47 uboot-env.dfu
-rw-r--r-- 1 root 7.7K  6月 11 17:47 uboot-env.txt
-rwxr-xr-x 1 root 4.4M  6月 11 17:35 zImage
-rw-r--r-- 1 root  23K  6月 11 17:37 zynq-AidoSDR.dtb

```



## Make SD card boot image

After the firmware building finished, you can build the SD card boot image for device. Just type the following command.

```sh
make sdimg
```

You will see the SD boot image in the build_sdimg folder. You can just  copy all these files in that folder into a SD card, plug the SD card  into the AidoSDR, set the jumper into SD card boot mode.

## Update Flash by DFU

DFU mode is available for AidoSDR, you can update the flash through DFU mode. Set the jumper to Flash Boot mode. After the device is powered on, press the DFU button, you will see both LED indicators in the device turn green, and now you can update the flash. You need to go to the build folder first, and then plug the Micro USB into the OTG port. Then, run the following command.

```sh
sudo dfu-util -a firmware.dfu -D ./AidoSDR.dfu
sudo dfu-util -a boot.dfu -D ./boot.dfu
sudo dfu-util -a uboot-env.dfu -D ./uboot-env.dfu
sudo dfu-util -a uboot-extra-env.dfu -U ./uboot-extra-env.dfu
```

Now you can repower device.



## Support 2r2t mode
If you want to use 2r2t mode, you need to enter the system and run the following command to write the mode configuradion into the nor flash. **But there is a little difference in sd card boot mode and qspi boot mode**

### QSPI mode
   ```sh
 fw_setenv attr_name compatible
 fw_setenv attr_val ad9361
 fw_setenv compatible ad9361
 fw_setenv mode 2r2t
 reboot
   ```

After restarting, use the command to detect whether the variable in the flash has been written. If the write is successful, then the 2r2t mode can be used.

Of course, thers is another way to configure the 2r2t mode, and use the command to write to the flash under uboot, such as

```sh
 setenv attr_name compatible
 setenv attr_val ad9361
 setenv compatible ad9361
 setenv mode 2r2t
 saveenv
 reset
```

 ### SD mode
 You need to modify some parameters in uEnv.txt file.

1. you need to modify the value of **adi_loadvals** as follows:

before fixing:
```txt
 adi_loadvals=fdt addr ${fit_load_address}......
```
after fixing:
 ```txt
 adi_loadvals=fdt addr ${devicetree_load_address}......
 ```

2. you need to modify the value of **mode** as follows:

before fixing:
```txt
maxcpus=1
mode=1r1t
```
after fixing:
```txt
maxcpus=1
mode=2r2t
```

3. you need to modify the value of **sdboot( add run adi_loadvals and #{fit_config})** as follows:

before fixing:
```txt
sdboot=if mmcinfo; then run uenvboot; echo Copying Linux from SD to RAM... && load mmc 0 ${fit_load_address} ${kernel_image} && load mmc 0 ${devicetree_load_address} ${devicetree_image} && load mmc 0 ${ramdisk_load_address} ${ramdisk_image} bootm ${fit_load_address} ${ramdisk_load_address} ${devicetree_load_address}; fi
```
after fixing:
```txt
sdboot=if mmcinfo; then run uenvboot; echo Copying Linux from SD to RAM... && load mmc 0 ${fit_load_address} ${kernel_image} && load mmc 0 ${devicetree_load_address} ${devicetree_image} && load mmc 0 ${ramdisk_load_address} ${ramdisk_image} && run adi_loadvals;bootm ${fit_load_address} ${ramdisk_load_address} ${devicetree_load_address}#{fit_config}; fi
```

4. you need to **add the following parameters(attr_name attr_val compatible)** in the last line:

before fixing:
```txt
usbboot=if usb start; then run uenvboot; echo Copying Linux from USB to RAM... && load usb 0 ${fit_load_address} ${kernel_image} && load usb 0 ${devicetree_load_address} ${devicetree_image} && load usb 0 ${ramdisk_load_address} ${ramdisk_image} && bootm ${fit_load_address} ${ramdisk_load_address} ${devicetree_load_address}; fi
```
after fixing:
```txt
usbboot=if usb start; then run uenvboot; echo Copying Linux from USB to RAM... && load usb 0 ${fit_load_address} ${kernel_image} && load usb 0 ${devicetree_load_address} ${devicetree_image} && load usb 0 ${ramdisk_load_address} ${ramdisk_image} && bootm ${fit_load_address} ${ramdisk_load_address} ${devicetree_load_address}; fi
attr_name=compatible
attr_val=ad9361
compatible=ad9361
```

Then you can enjoy the 2r2t mode.
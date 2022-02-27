# Arch-Linux-ARM-on-rpi-CM4
## This repository explains how to setup AArch 64 mainline kernel on Raspberry PI Compute Module 4

 </br>

AArch64 Installation is similar to [Raspberry Pi 4 installation](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4)

</br>

### Follow the official install manual for [Raspberry Pi 4 installation](https://archlinuxarm.org/platforms/armv8/broadcom/raspberry-pi-4) for the AArch64 Installation but
### **DON'T issue `sed -i 's/mmcblk0/mmcblk1/g' root/etc/fstab`** and DON'T UNMOUNT yet.

</br>

# Linux 5.16 kernel doesn't have the bcm2711-rpi-cm4.dtb so you can use the fixed precompiled [bcm2711-rpi-cm4.dtb](https://github.com/ivan95603/Arch-Linux-ARM-on-rpi-CM4/blob/main/predone/bcm2711-rpi-cm4.dtb) file from me or follow the directions below to compile it yourself.


</br>
    Install device tree compiler:

    On Debinan based base system:
    sudo apt-get install device-tree-compiler

</br>

### Download newest DTB from https://github.com/raspberrypi/firmware/blob/master/boot/bcm2711-rpi-cm4.dtb

 </br>
    Decompile the Device Tree File:
 </br>

    dtc -I dtb -O dts bcm2711-rpi-cm4.dtb -o bcm2711-rpi-cm4.dts 
</br>

### Change in the downloaded bcm2711-rpi-cm4.dts </br> **dma-ranges** according to the following [link](https://forums.raspberrypi.com/viewtopic.php?t=314845).

</br>

    emmc2bus: emmc2bus {
            compatible = "simple-bus";
            #address-cells = <2>;
            #size-cells = <1>;

            ranges = <0x0 0x7e000000  0x0 0xfe000000  0x01800000>;
            dma-ranges = <0x0 0xc0000000  0x0 0x00000000  0x40000000>;

            emmc2: mmc@7e340000 {
                compatible = "brcm,bcm2711-emmc2";
                reg = <0x0 0x7e340000 0x100>;
                interrupts = <GIC_SPI 126 IRQ_TYPE_LEVEL_HIGH>;
                clocks = <&clocks BCM2711_CLOCK_EMMC2>;
                status = "disabled";
            };
        };

### to


    dma-ranges = <0x00000000 0x00000000 0x00000000 0x00000000 0xfc000000>;
</br>

## Compile the dts file into dtb file for kernel:

    dtc -O dtb -o bcm2711-rpi-cm4.dtb bcm2711-rpi-cm4.dts

## Copy into device boot partition dtb file:

    cp bcm2711-rpi-cm4.dtb boot/dtbs/broadcom

</br>

## You can use your custrom device tree file but make sure to name it bcm2711-rpi-cm4.dtb because the bootloader passes that name to the loader.

</br>

## /etc/fstab file on device should look like this:

    [alarm@alarm ~]$ cat /etc/fstab 
    /dev/mmcblk0p1  /boot   vfat    defaults        0       0  <br/>
    /dev/mmcblk0p2  /       ext4    defaults,noatime  0       1


# **DON'T ISSUE** sed -i 's/mmcblk0/mmcblk1/g' root/etc/fstab it won't boot.

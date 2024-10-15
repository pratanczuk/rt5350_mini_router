

# Mini-USB-Portable-3G-4G-router---rt5350f
Imported from my old blog:
https://my-embedded.blogspot.com/2014/01/mini-usb-portable-3g4g-router-rt5350f.html

### Mini USB Portable 3G/4G router - rt5350f - 16M version


## Introduction

This repository provides a detailed guide on upgrading and hacking the Mini USB Portable 3G/4G WiFi Hotspot router based on the RT5350F SoC. The guide includes steps to:

-   Upgrade the bootloader to a functional version.
-   Install OpenWrt firmware for enhanced features.
-   Connect to the router via UART for advanced configurations.
-   Program the SPI flash without desoldering.
-   Upgrade the RAM from 16 MB to 32 MB.

  
**Note**: The hardware is similar to HAME A5 Mini and HAME A15 routers, but with different firmware.

Mini USB Portable 3G/4G WiFi Hotspot IEEE 802.11b/g/n 150Mbps Wireless Router - [search on ebay](http://www.ebay.com/sch/i.html?_odkw=3g+4g+wifi+router+mini&_osacat=0&_from=R40&_trksid=p2045573.m570.l1313.TR0.TRC0.XMini+USB+Portable+3G%2F4G+Wireless+Router&_nkw=Mini+USB+Portable+3G%2F4G+Wireless+Router&_sacat=0)  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjY5-vXJwhOE37Jdo0PsrWmtwdHSL_GpCtM3-1vqqHGv9CAAMHkfQTfpkoDpr_jvZHgQpfdm2u3ntgFwVktLeZf8iLQBATaabNwApfXdm8RrNQ7XnVE8EGIQLrXG6oQkFNeDVh0sIv3xtle/s200/mini_4g_case_top.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjY5-vXJwhOE37Jdo0PsrWmtwdHSL_GpCtM3-1vqqHGv9CAAMHkfQTfpkoDpr_jvZHgQpfdm2u3ntgFwVktLeZf8iLQBATaabNwApfXdm8RrNQ7XnVE8EGIQLrXG6oQkFNeDVh0sIv3xtle/s1600/mini_4g_case_top.jpg)


Upgrade with default firmware via WEB interface is not possible. Upgrade from u-boot (tftp) is not working too - because of broken ethernet configuration. Upgrade from u-boot via serial (kermit) will end with bricked router too ( verified).

  

Below you will find the way how to bring new working u-boot and openwrt firmware at once.

### Basic device parameters:

-   SoC RT5350
-   MIPS CPU
-   Clock 360 MHz
-   4 MByte Flash (GD25Q32)
-   16 MByte RAM (EM639165TS-6G)
-   USB Host 2.0
-   10/100 Ethernet switch
-   802.11n interface
-   I2C
-   Uart
-   Reset button
-   2 LEDs
-   Micro usb as power source

### GPIO description:

  

-   GPIO0 - Reset Button
-   GPIO1 - I2C_SD
-   GPIO2 - I2C_SCLK
-   GPIO7 - USB Power
-   GPIO12 - USB Root Hub Power
-   GPIO13 - Unknown, but seems to be used in bootloader
-   GPIO17 - Red Power LED
-   GPIO20 - Blue System LED

  

## Bootloader

Default u-boot bootloader is broken - ethernet is not fully initialized, and it seems that it is protected against starting other firmware than oryginal one.  
Do not try to upgrade this bootloader from bootloader menu - you will brick your device !  
  
To load openwrt firmware new bootloader is needed - I am using - [uboot128.img](https://github.com/pratanczuk/rt5350_mini_router/blob/main/img/uboot128.img) taken from [JiapengLi](https://github.com/JiapengLi/OpenWrt-HiLink-HLK-RM04/blob/master/image/uboot128.img)  
  
**Openwrt firmware**  
  

Default firmware cannot accept openwrt images so at the beggining (after u-boot upgrade) we will load minimal openwrt firmware [firmware.img](https://github.com/pratanczuk/rt5350_mini_router/blob/main/img/firmware.img) .  
  

**Upgrade procedure**  
  

Verification  

Check twice if it is router we are talking about:

  

-   Take a look on photos of PCB
-   Check MDT structure by call "cat /proc/mtd", has to be like below:

> cat /proc/mtd

> dev: size erasesize name  
>   
> mtd2: 00010000 00010000 "Config"  
> mtd3: 00010000 00010000 "Factory"  
> mtd4: 00117a6d 00010000 "Kernel"  
> mtd5: 00298593 00010000 "RootFS"  
> mtd6: 003b0000 00010000 "Kernel_RootFS"  
> mtd0: 00400000 00010000 "ALL"  
> mtd1: 00030000 00010000 "Bootloader"

> Think twice :)

#### First step

#### 

-   Reset router to default settings
-   Connect to router via telnet and login with default user name (admin) and password (admin)
-   Now you can upgrade router via FTP or USB

#### Upgrade via FTP

#### 

-   Release memory by commands:

> killall udhcpd  
> killall dnsmasq  
> killall nvram_daemon  
> rmmod ehci_hcd  
> rmmod ohci_hcd  
> ifconfig ra0 down  
> rmmod rt2860v2_ap

  

-   probably you will be disconnected after last command, so login via telnet again.
-   Verify free memory by calling free, check if you have more than 3500 free memory

  

> # free total used free shared buffers Mem: 12884 9180 3704 0 532 Swap: 0 0 0Total: 12884 9180 3704

-   Resize TMP, and start proftp server

> mount -o remount,size=4M tmpfs /tmp  
> proftpd.sh server 192.168.100.1 192.168.100.1 21 10  
> proftpd

-   Connect to router via ftp and copy uboot128.img and firmware.img to /tmp directory on router
-   !  Upgrade uboot - be careful, do not reset router during and after this operation !

> mtd_write write /tmp/uboot128.img Bootloader

-   You should see on console

> #Unlocking Bootloader ...  
> #Writing from /tmp/uboot128.img to Bootloader ... [w]

-   ! Upgrade firmware - do not reset router during this operation!

> mtd_write write /tmp/firmware.img Kernel_RootFS

-   You should see on console

> #Unlocking Kernel_RootFS ...  
> #Writing from /tmp/firmware.img to Kernel_RootFS ... [w]

-   Reboot router :), enjoy new u-boot with working ethernet and openwrt firmware. Now you can use standard openwrt upgrade procedure.

> reboot

-   Router will set IP address to 192.168.100.1 , you can login via telnet

  
Upgrade via USB

#### 

-   Prepare usb stick, format it with fat filesystem, and copy uboot128.img and firmware.img
-   Conect usb stick to router and mount it by:

> mount /dev/sda1 /mnt

-   Wait a few seconds and verify if you see files

> ls /mnt

-   You should see content, do not go further if you do not see files  !

> uboot128.img firmware.img

-   ! Upgrade uboot - be careful, do not reset router during and after this operation !

> mtd_write write /mnt/uboot128.img Bootloader

-   You should see on console

> #Unlocking Bootloader ...  
> #Writing from /mnt/uboot128.img to Bootloader ... [w]

-   ! Upgrade firmware - do not reset router during this operation!

> mtd_write write /mnt/firmware.img Kernel_RootFS

-   You should see on console

> #Unlocking Kernel_RootFS ...  
> #Writing from /mnt/firmware.img to Kernel_RootFS ... [w]

-   Reboot router :), enjoy new u-boot with working ethernet and openwrt firmware. Now you can use standard openwrt upgrade procedure.

> reboot

-   Router will set IP address to 192.168.100.1 , you can login via telnet

Connect UART

In order to have access to bootloader (u-boot) menu and functions we have to connect serial port to the router board. Pads are quite large so soldering is relatively easy.  
  
Serial port parameters:  
  

-   Speed: 57600
-   Data Bits: 8
-   Parity: None
-   Stop Bits: 1

**!** One problem has to be fixed - when you will connect adapter to RX in router and power on system will hangs. To fix it temporary you have first power on router and next quickly connect RX cable. To fix this issue permamently you have connect router RX and adapter TX by resistor (470 ohm - 1k ohm).  
  
Solder three thin wires to the router GND, TX, RX and connect:  

-   Router GND to adapter GND,
-   Router RX via 470 ohm resistor to adapter TX,
-   Router TX to adapter RX.

  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjst5zc3uBEsvKrNt7MP5YQuLQUkgaHeJI8iXoayrFju9tS44E_tK9NllI7yXZ-V6Etu_n576LFF7Q1hKjumKlCOeb03V7k3dgS0IJ27ln_zCSxqgX8n4BFg9kTpvsPb74ejqJUDCFgq4Dh/s320/mini_4g_router_pcb_side_uart.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjst5zc3uBEsvKrNt7MP5YQuLQUkgaHeJI8iXoayrFju9tS44E_tK9NllI7yXZ-V6Etu_n576LFF7Q1hKjumKlCOeb03V7k3dgS0IJ27ln_zCSxqgX8n4BFg9kTpvsPb74ejqJUDCFgq4Dh/s1600/mini_4g_router_pcb_side_uart.jpg)

Router pads description

  

To test connection:  
  

1.  Connect USB adapter to PC
2.  Start terminal and connect to port with proper parameters (putty [http://www.putty.org/](http://www.putty.org/) or TeraTerm Pro [http://www.ayera.com/teraterm/](http://www.ayera.com/teraterm/) )
3.  Power on router - you shoud see bootloader logs

  

### USB to TTL UART converters

My recomendation is to use:

-   CP2102
-   FT232
-   CH340

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEggWQY7TNEwg_Pcpx9yEdi2rIuxJWUmNCB7mHl250MPQ92INooXC3Hek_IQIixytJtVUOm48OfIXVfZhSCV1o5OtoAtv0sepiNOXUh81D16s6Ls8e7XGwx-0m9Kf3Lky_D2ikVwnDEo8lWR/s200/cp2102.JPG)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEggWQY7TNEwg_Pcpx9yEdi2rIuxJWUmNCB7mHl250MPQ92INooXC3Hek_IQIixytJtVUOm48OfIXVfZhSCV1o5OtoAtv0sepiNOXUh81D16s6Ls8e7XGwx-0m9Kf3Lky_D2ikVwnDEo8lWR/s1600/cp2102.JPG)

CP2102 based adapter  

  

and avoid if possible prolific PL2303 because of unstability ( maybe there are plenty of clones and prolific is not guilty - hard to say).  
  

## Program SPI flash

When you failed with u-boot upgrade you can program SPI flash without desoldering it.  
You need to connect thin wires to MISO, MOSI, CLK, CS, VCC and GND. I recomend to desolder VCC pin and lift it little bit, you will solder it again after programming. Disconnecting VCC prevent and distrubances from SoC or other components.  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjQjwvnpGnRAJRDhrEA2JxkUnymUzA5WLqw6P_VVlHp7J9JW7lrgqLD3IbbsdX7Z03oOSZSltXRJoSyVFq_nm2o_pWlcOtQehxVRWzHJJGOGyZeusc0cuKzKcBsoHG2W9ogb_XJ_YMb-xdy/s320/spi_flash.JPG)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjQjwvnpGnRAJRDhrEA2JxkUnymUzA5WLqw6P_VVlHp7J9JW7lrgqLD3IbbsdX7Z03oOSZSltXRJoSyVFq_nm2o_pWlcOtQehxVRWzHJJGOGyZeusc0cuKzKcBsoHG2W9ogb_XJ_YMb-xdy/s1600/spi_flash.JPG)

SPI flash pinout

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh_FhU5QdDG-k_05B2ocJKJE8N7mElOx4X3N46YAnVNB_4KAN24cX435AJcJGI5Ts0lRt7_xMCs8zKPpJgumtUDetupP5it_2Y69f2e-R7twPXhXzrWykDiXBINAnYEJCyQGv6TBtnB-JJU/s320/mini_4g_router_spi_uart_wires_2.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEh_FhU5QdDG-k_05B2ocJKJE8N7mElOx4X3N46YAnVNB_4KAN24cX435AJcJGI5Ts0lRt7_xMCs8zKPpJgumtUDetupP5it_2Y69f2e-R7twPXhXzrWykDiXBINAnYEJCyQGv6TBtnB-JJU/s1600/mini_4g_router_spi_uart_wires_2.jpg)

Wires connected to SPI flash,  
lifted VCC pin at the right corner soldered after programming.

  
I have used simple programmer made of  
  

-   atmega32 + serduino ported to atmega32 ( [http://flashrom.org/Serprog/Arduino_flasher](http://flashrom.org/Serprog/Arduino_flasher) )
-   Flashroom PC software [http://flashrom.org/Flashrom](http://flashrom.org/Flashrom) compiled with MinGW

  
But you can use any SPI programmer which supports GigaDevice GD25Q32 or flash you currently have.  
  

## Upgrade RAM

You can upgrade RAM by desoldering SDRAM chip and soldering chip with 32 MB. It is required to have 4M x 16bit x 4 (otherwise you would need to recompile bootloader) taken from old laptop memory PC133 SO-DIMM. It can be by example:  
  

-   Winbond W9825G6JH-6
-   Samsung K4S561632C-TC
-   Samsung K4S561632E

-   Micron mT48LC16m16A2
-   EtronTech EM63A165
-   ESMT M12L2561616A

According to Boot strapping description pins:  

-   EPHY_LED3_N
-   EPHY_LDE2_N

are used to determine SDRAM size:  

-   0,0: 2 MB/8 MB (default)
-   0,1: 8 MB/16 MB
-   1,0: 16 MB/32 MB, 32 MB*2
-   1,1: 32 MB

In order to set memory to 32MB you have to add 4.7k resistor (or just shorten)  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgqDcsMptLE4K-YB5NHFxTvCC6K3IpbDNPrBBKgKqFE0AgHcu6uqcbeWdudSsx7rwbB9zuClUso14dlbck8QkD1V39iIZ_JBIYiIVDvLJE_RYRdb4d5NQhUSJJrMe4q8TAFfbNYeuXOfLKF/s320/mini_4g_router_ram_upgrade.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgqDcsMptLE4K-YB5NHFxTvCC6K3IpbDNPrBBKgKqFE0AgHcu6uqcbeWdudSsx7rwbB9zuClUso14dlbck8QkD1V39iIZ_JBIYiIVDvLJE_RYRdb4d5NQhUSJJrMe4q8TAFfbNYeuXOfLKF/s1600/mini_4g_router_ram_upgrade.jpg)

Location of resistor pads to pull-up EPHY_LDE2_N

  
  

## Links

[https://openwrt.org/](https://openwrt.org/)  
[http://wiki.openwrt.org/toh/unbranded/a5-v11](http://wiki.openwrt.org/toh/unbranded/a5-v11)  
[https://github.com/sternlabs/RT5350F-cheap-router](https://github.com/sternlabs/RT5350F-cheap-router)  
[http://www.digitalinferno.com/wiki/Wiki.jsp?page=Mini3G4GUSBRouterOpenWrtExternalUSB](http://www.digitalinferno.com/wiki/Wiki.jsp?page=Mini3G4GUSBRouterOpenWrtExternalUSB)  
[http://lnxpps.de/openwrt/hame-mpra5/](http://lnxpps.de/openwrt/hame-mpra5/)  
[https://github.com/Squonk42/OpenWrt-RT5350](https://github.com/Squonk42/OpenWrt-RT5350)  
[https://github.com/JiapengLi/OpenWrt-RT5350](https://github.com/JiapengLi/OpenWrt-RT5350)  
[http://eko.one.pl/?p=openwrt-gpio2](http://eko.one.pl/?p=openwrt-gpio2)  
  

## Photos

  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgtBeuLxGf4idDTpny3r99bIeXz7YGVKaOumutjPs2aot48blJFt9WXofdFKy7T-45ylLtnH8tPQFCPTf4Zz30cnYH9741AYAD-9ZIL34g0sZZliaRbSTuEzuooxVOcPdeadyfp5zJxS0MD/s400/mini_4g_router_pcb_top.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgtBeuLxGf4idDTpny3r99bIeXz7YGVKaOumutjPs2aot48blJFt9WXofdFKy7T-45ylLtnH8tPQFCPTf4Zz30cnYH9741AYAD-9ZIL34g0sZZliaRbSTuEzuooxVOcPdeadyfp5zJxS0MD/s1600/mini_4g_router_pcb_top.jpg)

**PCB top side**  
  

  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiSSQwnpkcUQxFjjzh03afMfd2_VpdbV7ejYC8UFDKsUPlsxZCxkUNUvrrDgiZxG8YFFe85JrFeQItP8VQXTPSOyl4OGiRfK8okPXC1U795YSQQ-DbTs9VFIBl6rbC92ZiKPe4ltuTjCGvl/s400/mini_4g_router_pcb_bottom.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEiSSQwnpkcUQxFjjzh03afMfd2_VpdbV7ejYC8UFDKsUPlsxZCxkUNUvrrDgiZxG8YFFe85JrFeQItP8VQXTPSOyl4OGiRfK8okPXC1U795YSQQ-DbTs9VFIBl6rbC92ZiKPe4ltuTjCGvl/s1600/mini_4g_router_pcb_bottom.jpg)

**PCB bottom side**

  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhlFL0d1ZNq_b-snWMKCdqHbY0S_pUvYqNdG57sNDfZx3cPPr9KeokB3jHFR1or2CE593TOHb0WapJ_naC97vd4kMOMmQA45cHMBymqW1E9iTTk-0rNfMWkx9_JcA-_tsI-dmdM1qXLIfGR/s400/mini_4g_case_dissasembled.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhlFL0d1ZNq_b-snWMKCdqHbY0S_pUvYqNdG57sNDfZx3cPPr9KeokB3jHFR1or2CE593TOHb0WapJ_naC97vd4kMOMmQA45cHMBymqW1E9iTTk-0rNfMWkx9_JcA-_tsI-dmdM1qXLIfGR/s1600/mini_4g_case_dissasembled.jpg)

**Opened case**

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjah34xBZMiO1lbFW3eX4Rh9kpiO7X71XRTQcUqBmTkxIQDzOOn63_fAKqHQf_qLSzJxef9OiraErr3Ya_ls3XkCOKSJvL5PX3aIoBab8p4BPvx3iGcQ3riR5C6X6CRQP05JtTq0l67jaI6/s400/mini_4g_case_dissasembly.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjah34xBZMiO1lbFW3eX4Rh9kpiO7X71XRTQcUqBmTkxIQDzOOn63_fAKqHQf_qLSzJxef9OiraErr3Ya_ls3XkCOKSJvL5PX3aIoBab8p4BPvx3iGcQ3riR5C6X6CRQP05JtTq0l67jaI6/s1600/mini_4g_case_dissasembly.jpg)

**How to open case**

  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhEnLfTnIT6pW2iZfOVc-uWU1IAGO1e3AynXMhnXLX_eXvm8KYVJQYF9g6Nvn7Ht8mUeOFOUGLAzIVyvpJUx3u-DwXKnn6HEh9_Ms3ikxWKJAHyGoRiTxS8hwFrFMWyVbV73fh8-JDXSSNX/s400/mini_4g_case_top.jpg)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhEnLfTnIT6pW2iZfOVc-uWU1IAGO1e3AynXMhnXLX_eXvm8KYVJQYF9g6Nvn7Ht8mUeOFOUGLAzIVyvpJUx3u-DwXKnn6HEh9_Ms3ikxWKJAHyGoRiTxS8hwFrFMWyVbV73fh8-JDXSSNX/s1600/mini_4g_case_top.jpg)

**Router case**

  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgd3LzIJJHua7pVL679BhyW2MEKYvKzfOu-8IevIQtJp-El3sirJ5V7vAe3sqk2EL5D-qpy7cNiavNiGJj5oO1u6xHOrsIebJ1jnMw0XYsxSrtFze6yx4FP1ZoV6HRmk1qkqbuciBC5uD8f/s1600/IMG_5859%5B1%5D.JPG)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEgd3LzIJJHua7pVL679BhyW2MEKYvKzfOu-8IevIQtJp-El3sirJ5V7vAe3sqk2EL5D-qpy7cNiavNiGJj5oO1u6xHOrsIebJ1jnMw0XYsxSrtFze6yx4FP1ZoV6HRmk1qkqbuciBC5uD8f/s1600/IMG_5859%5B1%5D.JPG)

Desoldered CPU, vias visible, from anton.rad


### Mini USB Portable 3G/4G router - rt5350f - 32M version

  
Short description how to upgrade router with 32 MB on board.  
  
Source: [http://www.ebay.de/itm/400572464175?ssPageName=STRK:MEWNX:IT&_trksid=p3984.m1497.l2649](http://www.ebay.de/itm/400572464175?ssPageName=STRK:MEWNX:IT&_trksid=p3984.m1497.l2649)  
  
Body:  
  

[![](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjYYBdQ8lXaT_r49JxVTeMav75tcN-t5gIkJzW7KG71cEANlYdWO7j6xWXfBFOoNTbKpN09L-_rcGgargFPpRPZS2neqL8M1_bEXxWWRyAskXXFknDG8AVaeMXYAIdQg7TqWCgya-YwEn4Y/s1600/router32m.PNG)](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjYYBdQ8lXaT_r49JxVTeMav75tcN-t5gIkJzW7KG71cEANlYdWO7j6xWXfBFOoNTbKpN09L-_rcGgargFPpRPZS2neqL8M1_bEXxWWRyAskXXFknDG8AVaeMXYAIdQg7TqWCgya-YwEn4Y/s1600/router32m.PNG)

  
PCB is exactly the same, only resistor to set mem size to 32m has been added.  
  

## Bootloader

Default u-boot bootloader is not fully functional, and it seems that it is protected against starting other firmware than oryginal one.  
Do not try to upgrade this bootloader from bootloader menu - you will brick your device !  
  
To load openwrt firmware new bootloader is needed - I am using - [uboot256.img](https://github.com/pratanczuk/rt5350_mini_router/blob/main/img_32/uboot256.img) taken from [JiapengLi](https://github.com/JiapengLi/OpenWrt-HiLink-HLK-RM04/blob/master/image/uboot128.img)  
  
**Openwrt firmware**  
  

Default firmware cannot accept openwrt images so at the beggining (after u-boot upgrade) we will load minimal openwrt firmware [mini.bin](https://github.com/pratanczuk/rt5350_mini_router/blob/main/img_32/mini.bin) .

**Upgrade procedure**  
  

Verification  

Check twice if it is router we are talking about:

  

-   Take a look on photos of PCB
-   Check MDT structure by call "cat /proc/mtd", has to be like below:

> cat /proc/mtd

> dev: size erasesize name  
> mtd0: 00400000 00010000 "ALL"  
> mtd1: 00030000 00010000 "Bootloader"  
> mtd2: 00010000 00010000 "Config"  
> mtd3: 00010000 00010000 "Factory"  
> mtd4: 003b0000 00010000 "Kernel"  

Think twice :)  
  

#### First step

#### 

-   Reset router to default settings
-   Connect to router via telnet and login with default user name (admin) and password (admin)
-   Now you can upgrade router via FTP or USB

#### Upgrade via FTP

#### 

-   Cceck free memory
-     
    

-   Verify free memory by calling free, check if you have more than 3500 free memory

> # free total used free shared buffersMem: 28584 17128 11456 0 0Swap: 0 0 0Total: 28584 17128 11456  

-   Resize TMP, and start proftp server

> mount -o remount,size=4M tmpfs /tmp  
> proftpd.sh server 192.168.100.1 192.168.100.1 21 10  
> proftpd

-   Connect to router via ftp and copy uboot256.img and mini.bin to /tmp directory on router
-   ! Upgrade uboot - be careful, do not reset router during and after this operation !

> mtd_write write /tmp/uboot256.img Bootloader

-   You should see on console

> #Unlocking Bootloader ...  
> #Writing from /tmp/uboot256.img to Bootloader ... [w]

-   ! Upgrade firmware - do not reset router during this operation!

> mtd_write write /tmp/firmware.img Kernel

-   You should see on console

> #Unlocking Kernel ...  
> #Writing from /tmp/mini.bin to Kernel ... [w]

-   Reboot router :), enjoy new u-boot with working ethernet and openwrt firmware. Now you can use standard openwrt upgrade procedure.

> reboot

-   Router will set IP address to 192.168.100.1 , you can login via telnet

  
Upgrade via USB

#### 

-   Prepare usb stick, format it with fat filesystem, and copy uboot256.img and firmware.img
-   Conect usb stick to router and mount it by:

> mount /dev/sda1 /mnt

-   Wait a few seconds and verify if you see files

> ls /mnt

-   You should see content, do not go further if you do not see files !

> uboot256.img mini.bin

-   ! Upgrade uboot - be careful, do not reset router during and after this operation !

> mtd_write write /mnt/uboot256.img Bootloader

-   You should see on console

> #Unlocking Bootloader ...  
> #Writing from /mnt/uboot256.img to Bootloader ... [w]

-   ! Upgrade firmware - do not reset router during this operation!

> mtd_write write /mnt/mini.bin Kernel

-   You should see on console

> #Unlocking Kernel ...  
> #Writing from /mnt/mini.bin to Kernel ... [w]

-   Reboot router :), enjoy new u-boot with working ethernet and openwrt firmware. Now you can use standard openwrt upgrade procedure.

> reboot

-   Router will set IP address to 192.168.100.1 , you can login via telnet

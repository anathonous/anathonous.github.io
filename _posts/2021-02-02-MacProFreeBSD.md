---
layout: post
title: FreeBSD on Mac Pro 2006
categories: [FreeBSD]
---

Two years ago I recently installed FreeBSD 12.0 on a MacPro 1,1. Everything should still work on 12.2.

Due to the EFI being 32bit I had to use a website and tool to remove EFI functions from the installer.
(I have since updated my EFI to 2,1)

(https://mattgadient.com/2016/07/11/linux-dvd-images-and-how-to-for-32-bit-efi-macs-late-2006-models/) Worked great for creating an ISO. Use MBR bootloader during install. Be sure to install src and ports pkg.

There's a few things you have to configure to get it to work properly and there are still some small things that need to be tuned.

## Packages/src/ports (Security)
```
Change the word quarterly to latest in /etc/pkg/FreeBSD.conf
Then:
pkg update
pkg upgrade
freebsd-update fetch
freebsd-update install
portsnap fetch
portsnap extract
portsnap update
```

## Intel ucode
```
pkg install devcpu-data

/boot/loader.conf
cpu_microcode_load="YES"
cpu_microcode_name="/boot/firmware/intel-ucode.bin"
```

## Firewall
```
/etc/rc.conf
pf_enable="YES"
pf_rules="/etc/pf.conf"

/etc/pf.conf
block in all
pass out all keep state
```

## Audio
The default audio needs to be enabled and the port output needs to be switched. This will change it for line out on the back.
```
/etc/sysctl.conf 
hw.snd.default_unit=3
dev.hdaa.0.config="ovref" 
dev.hdaa.0.gpio_config="0=set 1=set" 
dev.hdaa.0.nid21_config="as=4 seq=15"
```
```
/etc/rc.conf 
sndiod_enable="YES"
```

## Video
I was able to get the video working by installing the old nvidia driver. The 304 release worked. "Nvidia GeForce 7300GT"
```
pkg install nvidia-driver-304 

/etc/rc.conf  
linux_enable="YES" 
linux64_enable="YES" 
nvidia_load="YES" 
nvidia-modeset_load="yes" 
linux_load="YES" 
```
```	  
/boot/loader.conf  
linux_enable="YES" 
linux64_enable="YES" 
nvidia_load="YES" 
nvidia-modeset_load="yes" 
linux_load="YES" 
agp_load="YES"
```
The video driver does act wonky in tty though. It will flash green and pink with the text going garbled and i noticed the text is still very large in tty. Changing you vt will resolve this issue.
```
/boot/loader.conf
kern.vty=sc
```

 ## Kernel
I get tons of kernel related errors. Apparently it has something to do with the smart battery. You can disable them.
```
/boot/loader.conf 
debug.acpi.disabled="smbat"
```
System Management Controller services are not enabled by default. You will need to compile a kernel with asmc support.
```
(https://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/kernelconfig-building.html)

You will need to download (https://svnweb.freebsd.org/base?view=revision&revision=342756)
and put them in 
/usr/src/sys/dev/asmc/

You want to make a copy (cp) of the GENERIC kernel. Say cp -r GENERIC MAC in /usr/src/sys/amd64/conf/ and then "make buildkernel KERNCONF=MAC" when compiling.

Add the following lines to your kernel conf. 

device          coretemp 
device          asmc 
device          smb 
device          smbus 
device          smbios 
device          cpuctl

and add to /boot/loader.conf
asmc_load="YES"
```
## Time

The clock is horrible off.
```
Code:
tzsetup  
Select Yes. Scroll down to bottom and select UTC.
```

That should be about it.

The FreeBSD Foundation has some great tutorials on installing a working desktop environment as well.

(https://www.freebsdfoundation.org/freebsd/how-to-guides/installing-a-desktop-environment-on-freebsd/)



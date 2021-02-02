# dualboot
Challenges and discussions for dual boot systems


Most recommended setup for dual boot (Windows + Linux):  
Install Windows first on your desktop/laptop. While installing Windows keep a unformatted partition of 20 GB or more for Linux distribution and a common partition of suitable size. If you have Windows pre-installed, and it is on single disk (HDD/SSD), shrink volume carefully to create space for Linux and the common partition.  
Install Linux (try Ubuntu) along side Windows. You should keep a common partition that is accessible or shared for both OS.  

[Dual Boot.pdf](https://github.com/rks101/dualboot/blob/main/Dual%20Boot.pdf) has listed out some steps. Credits: Ankit Gupta  

**About Virtual install**  
If you are scared of dual boot or not sure where it leads to, you can install Linux on a VM using some virtualization software such as Oracle Virtual Box or VMware WorkStation. Once you get familiar and hands-on with Linux, you can dual boot your system.  

Even till 2020, dual boot systems are not shipped. There is no reason to worry about dual boot in 2020 as Ubuntu and other distribution communities have fairly good support for this.   


**On BIOS v/s UEFI**  
Slowly, Legacy BIOS+MBR mode boot has moved to UEFI+GPT mode.  
[UEFI](https://help.ubuntu.com/community/UEFI) - Unified Extensible Firmware Interface (UEFI) is the next generation of BIOS firmware and will eventually replace Legacy BIOS. This page on Ubuntu community gives some insights into UEFI booting, dual boot and secure boot.  

A 5 minute [story](https://www.freecodecamp.org/news/mbr-vs-gpt-whats-the-difference-between-an-mbr-partition-and-a-gpt-partition-solved/) on partitions, and partition tables.  

----

How can we see partition table using command line?  
```
$ fdisk -l 
............
Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: WDC WD10SPZX-75Z
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt
Disk identifier: 082Z74ZF-EZ78-4Z77-9ZD1-6Z110Z47CFZE

Device          Start        End    Sectors   Size Type
/dev/sda1        2048     923647     921600   450M Windows recovery environment
/dev/sda2      923648    1128447     204800   100M EFI System
/dev/sda3     1128448    1161215      32768    16M Microsoft reserved
/dev/sda4     1161216  306162323  305001108 145.4G Microsoft basic data
/dev/sda5   306163712  307199999    1036288   506M Windows recovery environment
/dev/sda6   307202048 1331202047 1024000000 488.3G Microsoft basic data
/dev/sda7  1331202048 1393702911   62500864  29.8G Linux swap
/dev/sda8  1393702912 1953523711  559820800   267G Linux filesystem
```
In the output of fdisk, llok for **Disk /dev/sda** ignore entries listed for loop devices.  

**Sync clock across boots**  
After you have rebuilt system by installing multiple oeprating systems, and if clock is out of sync across different OS boots, you can sync the clock by setting "RTC in local TZ" to yes using the following:  

```
timedatectl set-local-rtc 1 --adjust-system-clock

timedatectl
```

----

**System not booting to Linux, only booting to Windows**  

After dual boot (Windows and Linux), if the system is booting to Windows OS only, you should check two things first.  
 (i) BIOS: see if Windows Boot manager (single boot loader) is selected to boot  
(ii) Fast Startup set in Windows => Control Panel => Power Options: you should uncheck Fast Startup option. With this, the system performs a clean cold boot across reboots and dual boot loader (GRUB/LILO) wakes up to give you options and you can select the OS to boot with.  

Apart from these two, it is possible that boot loader did not install correctly. Also, this may happen if you had to reinstall Windows post dual boot.  
You should repair boot loader.  

----

**How to repair Boot Loader**

You can check [Boot Repair](https://www.howtogeek.com/114884/how-to-repair-grub2-when-ubuntu-wont-boot/) utiity with GUI. Using bootable USB drive, boot your system and install Boot Repair. Here is another tutorial to [repair GRUB](https://linuxhint.com/ubuntu_boot_repair_tutorial/) for most of the times.  

----

# dualboot
Discussions for dual boot systems.    

   * [Dualboot](#dualboot)
      * [Recommended Dual Boot Setup](#recommended-dual-boot-setup)
      * [On MBR vs GPT](#on-mbr-vs-GPT)
      * [On BIOS vs UEFI](#on-bios-vs-uefi)
      * [Virtual Install](#virtual-install)
      * [Partition Table](#partition-table)
      * [Sync clock across boots](#sync-clock-across-boots)
      * [Locked or read-only Windows partition or locked NTFS partition](#locked-or-read--only-Windows-partition-or-locked-NTFS-partition)
      * [System not booting to Linux or only booting to Windows](#system-not-booting-to-Linux-or-only-booting-to-Windows)
      * [How to Repair Boot Loader](#how-to-repair-boot-loader)

## Recommended Dual Boot Setup   
The most recommended setup for dual boot is Windows + Linux (Ubuntu/Fedora/RedHat/Mint).   

Install Windows first on your desktop/laptop. While installing Windows, keep an unformatted partition of 30 GB or more for Linux distribution and a shared data partition of suitable size. If you have Windows pre-installed, and it is on a single disk (HDD/SSD), shrink the volume carefully using the Computer Management utility to create space for Linux and the shared partition.  
Install Linux (try Ubuntu) alongside Windows. You should keep a shared partition that is accessible or shared for both OSes.  

[Dual Boot.pdf](https://github.com/rks101/dualboot/blob/main/Dual%20Boot.pdf) has listed out some steps. Credits: Ankit Gupta     

[Install Windows 10 on Mac with Boot Camp Assistant](https://support.apple.com/en-in/HT201468)    

## On MBR vs GPT    
A HDD/SSD/SCSI disk should be partitioned (to define disk structure) and formatted (to create file system) before it can be used.    

Master Boot Record (MBR) and GUID Partition Table (GPT) are two partitioning styles.   

MBR allowed only four primary partitions on a disk and worked with disk sizes up to 2TB. MBR had a peculiar limitation. MBR allowed having a boot sector (for using the first boot loader) in the initial geometry of the disk (say first 1024 cylinders). Now, if this information is overwritten, you need to recreate or "repair" boot loader information. MBR was dependent on BIOS.     

When using GPT, these limitations are mainly dependent on the Operating system and file system. Windows may allow 128 partitions and much larger file sizes. GPT stores a random string for each disk partition, and the boot-related details are spread across disk geometry, having multiple copies. GPT relies on UEFI.    

----

## On BIOS vs UEFI  

Slowly, Legacy BIOS+MBR mode boot has moved to UEFI+GPT mode.  
[UEFI](https://help.ubuntu.com/community/UEFI) - Unified Extensible Firmware Interface (UEFI) is the next generation of BIOS firmware and will eventually replace Legacy BIOS. This page on the Ubuntu community gives some insights into UEFI booting, dual boot, and secure boot.  

A 5 minute [story](https://www.freecodecamp.org/news/mbr-vs-gpt-whats-the-difference-between-an-mbr-partition-and-a-gpt-partition-solved/) on partitions, and partition tables.  

----

## Virtual Install   
If you are unsure of dual boot, you can install Linux on a VM using some virtualization software such as Oracle Virtual Box or VMware Workstation. Once you get familiar and hands-on with Linux, you can dual-boot your system.   

Even till 2024, dual boot systems are not shipped. There is no reason to worry about dual boot, as Ubuntu and other distribution communities have good support for this.   

----

## Partition Table   
How can we see the partition table using the command line?   

Note:- This output shows a nice example of disk virtualisation - a single physical disk /dev/sda of 1000204886016 bytes partitioned into multiple virtual disks/drives/volumes/partitions/LPARs.    

```
$ sudo fdisk -l 
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
/dev/sda4     1161216  306162323  305001108 145.4G Microsoft basic data                 <= Windows partition, NTFS 
/dev/sda5   306163712  307199999    1036288   506M Windows recovery environment
/dev/sda6   307202048 1331202047 1024000000 488.3G Microsoft basic data                 <= NTFS, accessible from both OS 
/dev/sda7  1331202048 1393702911   62500864  29.8G Linux swap                           <= Linux Swap 
/dev/sda8  1393702912 1953523711  559820800   267G Linux filesystem                     <= Linux everyday partition 
```
In the output of fdisk, look for **Disk /dev/sda** and ignore entries listed for loop devices.   

----

## Sync clock across boots  
After you have rebuilt the system by installing multiple operating systems, and if the clock is out of sync across different OS boots, you can sync the clock by setting "RTC in local TZ" to yes using the following:  

```
timedatectl set-local-rtc 1 --adjust-system-clock

timedatectl
```
---- 

## Locked or read-only Windows partition or locked NTFS partition    

Sometimes a particular partition cannot be accessed or it is read-only, no write or update is alowed in a dual booted system with Windows. An unclean shutdown from the Windows filesystem or hibernation or fast boot enabled in Windows or incomplete update possibly gone wrong or a similar event can render an NTFS/FAT partition exclusively locked or read-only or inaccessible.   

Trying to access NTFS partition using mount, see the message:   
```
rps@Latitude-3490:~$ sudo mkdir /mnt/windows
rps@Latitude-3490:~$ sudo mount -t ntfs /dev/sda6 /mnt/shared
The disk contains an unclean file system (0, 0).                        <=
Metadata kept in Windows cache, refused to mount.
Falling back to read-only mount because the NTFS partition is in an
unsafe state. Please resume and shutdown Windows fully (no hibernation  <= 
or fast restarting.)                                                    <=
Could not mount read-write, trying read-only
```
Another attemp:   
```
rps@Latitude-3490:~$ sudo -i
[sudo] password for rps: 
root@Latitude-3490:~# mount -t ntfs -o rw /dev/sda6 /mnt/shared
Mount is denied because the NTFS volume is already exclusively opened.  <=
The volume may be already mounted, or another software may use it which <=
could be identified for example by the help of the 'fuser' command.
[1]+  Done                    $PANGPA start
root@Latitude-3490:~# id
uid=0(root) gid=0(root) groups=0(root)
root@Latitude-3490:~# chown 1000:1000 /mnt/shared
chown: changing ownership of '/mnt/shared': Read-only file system
root@Latitude-3490:~# 

```
See the output of parted below, fdisk is listed above of the same system for reference:  
```
rps@Latitude-3490:~$ sudo parted -l
Model: ATA WDC WD10SPZX-75Z (scsi)
Disk /dev/sda: 1000GB
Sector size (logical/physical): 512B/4096B
Partition Table: gpt
Disk Flags: 

Number  Start   End     Size    File system     Name                          Flags
 1      1049kB  473MB   472MB   ntfs            Basic data partition          hidden, diag
 2      473MB   578MB   105MB   fat32           EFI system partition          boot, esp
 3      578MB   595MB   16.8MB                  Microsoft reserved partition  msftres
 4      595MB   157GB   156GB   ntfs            Basic data partition          msftdata
 5      157GB   157GB   531MB   ntfs                                          hidden, diag
 6      157GB   682GB   524GB   ntfs            Basic data partition          msftdata
 7      682GB   714GB   32.0GB  linux-swap(v1)                                swap
 8      714GB   1000GB  287GB   ext4
```

TODO: Solving this read-only (or sometimes disappeared partition in GUI) situation is not trivial, and it may be irrecoverable. At worst: you may need to reinstall Windows at the cost of loosing some files. Before reinstall, there are certain steps to try. It is possible, none of them work and it happened with me twice in the last 20 years.    

* [Mount NTFS partition in Linux to backup](https://phoenixnap.com/kb/mount-ntfs-linux)   
* [Recovery options in Windows](https://support.microsoft.com/en-us/windows/recovery-options-in-windows-31ce2444-7de3-818c-d626-e3b5a3024da5)   
----

## System not booting to Linux or only booting to Windows   

After dual boot (Windows and Linux), if the system is booting to Windowcertains OS only, you should check two things first.  
 (i) BIOS: check if Windows Boot manager (single boot loader) is selected to boot or sequence boot loaders with first boot loader from linux (dual boot loader). GRUB allows chaining boot loaders.   
(ii) Fast Startup set in Windows => Control Panel => Power Options: you should uncheck Fast Startup option. With this, the system performs a clean cold boot across reboots and dual boot loader (GRUB/LILO) wakes up to give you options and you can select the OS to boot with.  

Apart from these two, it is possible that boot loader did not install correctly. Also, this may happen if you had to reinstall Windows post dual boot. You should repair boot loader.   

Before you reinstall Windows, try auto repair / reset PC / uninstall recent feature update / system restore to last safe point. One of this may work or none of them may work. If you are keeping this os, you may disable auto updates.   

----

## How to Repair Boot Loader   

You can check [Boot Repair](https://www.howtogeek.com/114884/how-to-repair-grub2-when-ubuntu-wont-boot/) utiity with GUI. Using bootable USB drive, boot your system and install Boot Repair. Here is another tutorial to [repair GRUB](https://linuxhint.com/ubuntu_boot_repair_tutorial/) for most of the times.  
Despite all these, sometimes things happen we hope they do not happen. Dual boot systems are not shipped and they are not tested for dual boot normally. Yet all that magic works. Do not loose heart, do not shout at people, if system crashes and you cannot recover. Rebuild it.    

Recently, I lost some data related to my PhD and work after a keyboard repair. Locked NVram detected on Ubuntu and boot-repair could not repair. I fact it was not recognizing Ubuntu boot itself, partition remained read-only. I had to rebuild system.   

----

## Password recovery options   
[Win 10](https://www.top-password.com/knowledge/forgot-windows-10-local-administrator-password.html) 
and [another link](https://www.wimware.com/how-to/reset-windows-10-enterprise-password.html)   

----

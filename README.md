# dualboot
Challenges and discussions for dual boot systems


Most recommended setup for dual boot (Windows + Linux):  
Install Windows first on your desktop/laptop. While installing Windows keep a unformatted partition of 20 GB or more for Linux distribution and a common partition of suitable size. If you have Windows pre-installed, and it is on single disk (HDD/SSD), shrink volume carefully to create space for Linux and the common partition.  
Install Linux (try Ubuntu) along side Windows. You should keep a common partition that is accessible or shared for both OS.  

[Dual Boot.pdf](https://github.com/rks101/dualboot/blob/main/Dual%20Boot.pdf) has listed out some steps. Credits: Ankit Gupta  

[Install Windows 10 on Mac with Boot Camp Assistant](https://support.apple.com/en-in/HT201468)  

**About Virtual install**  
If you are scared of dual boot or not sure where it leads to, you can install Linux on a VM using some virtualization software such as Oracle Virtual Box or VMware WorkStation. Once you get familiar and hands-on with Linux, you can dual boot your system.  

Even till 2020, dual boot systems are not shipped. There is no reason to worry about dual boot in 2020 as Ubuntu and other distribution communities have fairly good support for this.   


**On BIOS v/s UEFI**  
Slowly, Legacy BIOS+MBR mode boot has moved to UEFI+GPT mode.  
[UEFI](https://help.ubuntu.com/community/UEFI) - Unified Extensible Firmware Interface (UEFI) is the next generation of BIOS firmware and will eventually replace Legacy BIOS. This page on Ubuntu community gives some insights into UEFI booting, dual boot and secure boot.  

A 5 minute [story](https://www.freecodecamp.org/news/mbr-vs-gpt-whats-the-difference-between-an-mbr-partition-and-a-gpt-partition-solved/) on partitions, and partition tables.  

----

How can we see partition table using command line?   

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
In the output of fdisk, llok for **Disk /dev/sda** ignore entries listed for loop devices.  

**Sync clock across boots**  
After you have rebuilt system by installing multiple oeprating systems, and if clock is out of sync across different OS boots, you can sync the clock by setting "RTC in local TZ" to yes using the following:  

```
timedatectl set-local-rtc 1 --adjust-system-clock

timedatectl
```
---- 

**Locked or read-only Windows partition or a locked NTFS partition**    

Sometimes a certain partition cannot be accessed or it is read-only, no write or update is alowed in a dual booted system with Windows. An unclean shutdown from Windows filesystem or hibernation or fast boot enabled in Windows or incomplete update possibly gone wrong or similar event can render an NTFS/FAT partition exclusively locked or read-only or inaccessible.   

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

**System not booting to Linux, only booting to Windows**  

After dual boot (Windows and Linux), if the system is booting to Windows OS only, you should check two things first.  
 (i) BIOS: check if Windows Boot manager (single boot loader) is selected to boot or sequence boot loaders   
(ii) Fast Startup set in Windows => Control Panel => Power Options: you should uncheck Fast Startup option. With this, the system performs a clean cold boot across reboots and dual boot loader (GRUB/LILO) wakes up to give you options and you can select the OS to boot with.  

Apart from these two, it is possible that boot loader did not install correctly. Also, this may happen if you had to reinstall Windows post dual boot. You should repair boot loader.  

----

**How to repair Boot Loader**

You can check [Boot Repair](https://www.howtogeek.com/114884/how-to-repair-grub2-when-ubuntu-wont-boot/) utiity with GUI. Using bootable USB drive, boot your system and install Boot Repair. Here is another tutorial to [repair GRUB](https://linuxhint.com/ubuntu_boot_repair_tutorial/) for most of the times.  

----

Password recovery options: 
[Win 10[(https://www.top-password.com/knowledge/forgot-windows-10-local-administrator-password.html)
https://www.wimware.com/how-to/reset-windows-10-enterprise-password.html

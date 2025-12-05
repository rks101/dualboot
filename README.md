# dualboot
Discussions for dual boot systems.    

   * [Dualboot](#dualboot)
      * [Recommended Dual Boot Setup](#recommended-dual-boot-setup)
      * [On MBR vs GPT](#on-mbr-vs-GPT)
      * [On BIOS vs UEFI](#on-bios-vs-uefi)
      * [Virtual Install](#virtual-install)
      * [Power on to OS Desktop or Console](#power-on-to-os-desktop-or-console)
      * [Partition Table](#partition-table)
      * [Sync clock across boots](#sync-clock-across-boots)
      * [Locked or read-only Windows partition or locked NTFS partition](#locked-or-read-only-Windows-partition-or-locked-NTFS-partition)
      * [System not booting to Linux or only booting to Windows](#system-not-booting-to-Linux-or-only-booting-to-Windows)
      * [Entending EFI system partition](#extending-efi-system-partition)
      * [Firmware Update](#firmware-update)
      * [How to Repair Boot Loader](#how-to-repair-boot-loader)

## Recommended Dual Boot Setup   
The most recommended setup for dual boot is Windows (first) + Linux (Ubuntu/Fedora/RedHat/Mint) (second). Windows OS has a single boot loader, knows Windows realm. Linux distributions offer a dual boot loader - GRUB or LILO that knows both Windows and Linux realms (multiple file systems).    

Install Windows first on your desktop/laptop. While installing Windows, keep an unformatted partition of 30 GB or more for Linux distribution and a shared data partition of suitable size. If you have Windows pre-installed, and it is on a single disk (HDD/SSD), shrink the volume carefully using the Computer Management utility to create space for Linux and the shared partition.  
Install Linux (try Ubuntu) alongside Windows. You should keep a shared partition that is accessible or shared for both OSes.   

[Dual Boot.pdf](https://github.com/rks101/dualboot/blob/main/Dual%20Boot.pdf) has listed out some steps. Credits: Ankit Gupta     

People buy Mac and can install Windows later :) [Install Windows 10 on Mac with Boot Camp Assistant](https://support.apple.com/en-in/HT201468)    

----

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

## Power on to OS Desktop or Console   

Q. What steps are involved from Power on to getting an OS Desktop or console login?     

Correct this sequence: OS Image, Boot Loader, BIOS/UEFI, Power On, OS Desktop/console login, User Apps, OS Sub-systems,    

A. Power On to User applications:     
- Power On, run POST (Power On Self Test) checks (is power there, is battery installed, are compute/memory/network/store there, etc.),    
- BIOS/UEFI wakes up, knows boot sequence settings and boot loader,       
- Boot Loader / Boot Loader Chain tells where to get OS image from,   
- OS Image is loaded,    
- Loading Sub-systems (power, file system, memory, network, audio, GUI, long list of drivers),   
- OS Desktop/console login    
- User applications   

[Boot Process - a long story](https://www.youtube.com/watch?v=Masm_ec0JiQ&t=204s)     

----

## Partition Table   
How can we see the partition table using the command line?   

Note:- This output shows a nice example of disk virtualisation - a single physical disk /dev/sda of 1000204886016 bytes partitioned into multiple virtual disks/drives/volumes/partitions/LPARs.    

```
$ sudo fdisk -l 
............
Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors                     <= a SATA HDD 
Disk model: WDC WD10SPZX-75Z
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 4096 bytes
I/O size (minimum/optimal): 4096 bytes / 4096 bytes
Disklabel type: gpt                                                                    <= partition style 
Disk identifier: 082Z74ZF-EZ78-4Z77-9ZD1-6Z110Z47CFZE

Device          Start        End    Sectors   Size Type
/dev/sda1        2048     923647     921600   450M Windows recovery environment
/dev/sda2      923648    1128447     204800   100M EFI System                          <= EFI/UEFI (replaced BIOS), /boot is mounted here 
/dev/sda3     1128448    1161215      32768    16M Microsoft reserved
/dev/sda4     1161216  306162323  305001108 145.4G Microsoft basic data                 <= Windows partition, NTFS 
/dev/sda5   306163712  307199999    1036288   506M Windows recovery environment
/dev/sda6   307202048 1331202047 1024000000 488.3G Microsoft basic data                 <= NTFS, accessible from both OS 
/dev/sda7  1331202048 1393702911   62500864  29.8G Linux swap                           <= Linux Swap 
/dev/sda8  1393702912 1953523711  559820800   267G Linux filesystem                     <= Linux everyday partition 
```
In the output of fdisk, look for **Disk /dev/sda** and ignore entries listed for loop devices.   

For Solid State Drives (SSD), you will see entries as **/dev/nvme**    

```
Disk /dev/nvme0n1: 476.94 GiB, 512110190592 bytes, 1000215216 sectors
Disk model: BC711 NVMe SK hynix 512GB               
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 90AC9BEA-BE65-4C42-8C6D-4CE4C6349709

Device             Start        End   Sectors   Size Type
/dev/nvme0n1p1      2048     514047    512000   250M EFI System                           <= EFI/UEFI partition, /boot is mounted here, contains boot info  
/dev/nvme0n1p2   1562624    1824767    262144   128M Microsoft reserved
/dev/nvme0n1p3   1824768  483047423 481222656 229.5G Microsoft basic data                 <= Windows partition, NTFS file system 
/dev/nvme0n1p4 995047424  997257215   2209792   1.1G Windows recovery environment
/dev/nvme0n1p5 997261312 1000187903   2926592   1.4G Windows recovery environment
/dev/nvme0n1p6 483047424  547047423  64000000  30.5G Linux swap                           <= Linux Swap 
/dev/nvme0n1p7 547047424  995047423 448000000 213.6G Linux filesystem                     <= Linux everyday partition, ext4 file system 

Partition table entries are not in disk order.

```

----

## Sync clock across boots  
After you have rebuilt the system by installing multiple operating systems, and if the clock is out of sync across different OS boots, you can sync the clock by setting "RTC in local TZ" to yes using the following:  

```
timedatectl set-local-rtc 1 --adjust-system-clock
```
```
$ timedatectl 
               Local time: Tue 2020-08-05 19:59:10 IST
           Universal time: Tue 2020-08-05 14:29:10 UTC
                 RTC time: Tue 2020-08-05 14:29:10
                Time zone: Asia/Kolkata (IST, +0530)
System clock synchronized: yes                          <== notice yes for synched clock across OS boot
              NTP service: active
          RTC in local TZ: no
```
---- 

## Locked or read only Windows partition or locked NTFS partition    

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

## Extending EFI system partition    

EFI system partition (ESP) by Windows 11 may be too small for dual boot or Firmware Update. Here is a detailed discussion on [resizing ESP](https://superuser.com/questions/1230741/how-to-resize-the-efi-system-partition).    

----

## Firmware Update 

```
sudo fwupdmgr get-devices   

sudo fwupdmgr get-updates   

sudo fwupdmgr update   
```

```
$sudo fwupdmgr get-devices   

Dell Inc. Latitude 5420
│
├─11th Gen Intel Core™ i5-1135G7 @ 2.40GHz:
│     Device ID:          3027
│     Current version:    0x000000bc
│     Vendor:             Intel
│     GUIDs:              1234 ← CPUID\PRO_0&FAM_06&MOD_8C
│                         1df9 ← CPUID\PRO_0&FAM_06&MOD_8C&STP_1
│     Device Flags:       • Internal device
│   
├─BC711 NVMe SK hynix 512GB:
│     Device ID:          1317
│     Summary:            NVM Express solid state drive
│     Current version:    41002131
│     Vendor:             SK hynix (NVME:0x1C5C)
│     Install Duration:   6 seconds
│     Serial Number:      3A07
│     Update State:       Needs reboot         <== Restart required status 
│     Last modified:      2024-12-05 10:20
│     GUIDs:              3334 ← STORAGE-DELL-110001
│                         0003
│     Device Flags:       • Updatable
│                         • System requires external power source
│                         • Supported on remote server
│                         • Needs a reboot after installation
│                         • Signed Payload
│   
├─System Firmware:
│ │   Device ID:          71e98
│ │   Summary:            UEFI System Resource Table device (updated via NVRAM)
│ │   Current version:    1.45.0
│ │   Minimum Version:    1.45.0
│ │   Vendor:             Dell (DMI:Dell Inc.)
│ │   Update State:       Failed              <== Should be Success 
│ │   Update Error:       /boot/efi does not have sufficient space, required 73.5 MB, got 22.6 MB 
│ │   Last modified:      2024-12-05 10:21
│ │   GUID:               1091
│ │   Device Flags:       • Internal device
│ │                       • Updatable
│ │                       • System requires external power source
│ │                       • Supported on remote server
│ │                       • Needs a reboot after installation
│ │                       • Cryptographic hash verification is available
│ │                       • Device is usable for the duration of the update
│ │   Device Requests:    • Message
│ │ 
│ ├─BootGuard Configuration:
│ │     Device ID:        80753
│ │     Current version:  20
│ │     Vendor:           Intel Corporation (MEI:0x8086)
│ │     GUIDs:            ec65
│ │                       0f70 ← MEI\VEN_8086&DEV_A0E0
│ │                       6d76 ← MEI\VEN_8086&DEV_A0E0&SUBSYS_10280A20
│ │     Device Flags:     • Internal device
│ │   
│ └─UEFI dbx:
│       Device ID:        a590
│       Summary:          UEFI revocation database
│       Current version:  20241101
│       Minimum Version:  20241101
│       Vendor:           UEFI:Microsoft
│       Install Duration: 1 second
│       GUIDs:            cca7 ← UEFI\CRT_SOMETHING&ARCH_X64
│                         9731 ← UEFI\CRT_SOMETHING&ARCH_X64
│       Device Flags:     • Internal device
│                         • Updatable
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│                         • Only version upgrades are allowed
│                         • Signed Payload
│     
├─TPM:
│     Device ID:          34d6
│     Current version:    1.258.0.0
│     Vendor:             ST Microelectronics (TPM:STM)
│     GUIDs:              4c32 ← TPM\VEN_STM&DEV_0001
│                         e918 ← TPM\VEN_STM&MOD_
│                         f319 ← TPM\VEN_STM&DEV_0001&VER_2.0
│                         3c48 ← TPM\VEN_STM&MOD_&VER_2.0
│                         cf70 ← 0a20-2.0
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device can recover flash failures
│                         • Full disk encryption secrets may be invalidated when updating
│                         • Signed Payload
│   
├─TigerLake-LP GT2 [Iris Xe Graphics]:
│     Device ID:          e00a
│     Current version:    01
│     Vendor:             Intel Corporation (PCI:0x8086)
│     GUIDs:              aa53 ← PCI\VEN_8086&DEV_9A49
│                         63df ← PCI\VEN_8086&DEV_9A49&SUBSYS_10280A20
│     Device Flags:       • Internal device
│                         • Cryptographic hash verification is available
│   
├─UEFI Device Firmware:
│     Device ID:          409d
│     Summary:            UEFI System Resource Table device (updated via NVRAM)
│     Current version:    188
│     Minimum Version:    188
│     Vendor:             DMI:Dell Inc.
│     Update State:       Success
│     GUID:               ecd0
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│     Device Requests:    • Message
│   
├─UEFI Device Firmware:
│     Device ID:          e307
│     Summary:            UEFI System Resource Table device (updated via NVRAM)
│     Current version:    1090527537
│     Minimum Version:    1090527537
│     Vendor:             DMI:Dell Inc.
│     Update State:       Success
│     GUID:               003
│     Device Flags:       • Internal device
│                         • Updatable
│                         • System requires external power source
│                         • Needs a reboot after installation
│                         • Device is usable for the duration of the update
│     Device Requests:    • Message
│   
────────────────────────────────────────────────
Devices that were not updated correctly:
 • System Firmware (1.45.0 → 1.49.0)
Devices that have been updated successfully:
 • System Firmware (1.34.1 → 1.39.1)

```

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

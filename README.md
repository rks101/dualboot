# dualboot
Challenges and discussions for dual boot systems


Most recommended setup for dual boot (Windows + Linux):  
Install Windows first on your desktop/laptop. While installing Windows keep a unformatted partition of 20 GB or more for Linux distribution and a common partition of suitable size. If you have Windows pre-installed, and it is on single disk (HDD/SSD), shrink volume carefully to create space for Linux and the common partition.  
Install Linux (try Ubuntu) along side Windows. You should keep a common partition that is accessible or shared for both OS.  


**About Virtual install**  
If you are scared of dual boot or not sure where it leads to, you can install Linux on a VM using some virtualization software such as Oracle Virtual Box or VMware WorkStation. Once you get familiar and hands-on with Linux, you can dual boot your system.  

Even till 2020, dual boot systems are not shipped. There is no reason to worry about dual boot in 2020 as Ubuntu and other distribution communities have fairly good support for this.   


**On BIOS v/s UEFI**  
Slowly, Legacy BIOS+MBR mode boot has moved to UEFI+GPT mode.  
[UEFI](https://help.ubuntu.com/community/UEFI) - Unified Extensible Firmware Interface (UEFI) is the next generation of BIOS firmware and will eventually replace Legacy BIOS. This page on Ubuntu community gives some insights into UEFI booting, dual boot and secure boot.  

A 5 minute [story](https://www.freecodecamp.org/news/mbr-vs-gpt-whats-the-difference-between-an-mbr-partition-and-a-gpt-partition-solved/) on partitions, and partition tables. 

**Sync clock across boots**  
After you have rebuilt system by installing multiple oeprating systems, and if clock is out of sync across different OS boot, you can sync the clock using:  

```
timedatectl set-local-rtc 1 --adjust-system-clock

timedatectl
```

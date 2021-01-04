# dualboot
Challenges and remedies for dual boot systems


Most recommended setup for dual boot:  
Install Windows first on desktop/laptop. While installing Windows keep a unformatted partition of 20 GB or more for Linux distribution and a common partition of suitable size. If you have Windows pre-installed, and it is on single disk (HDD/SSD), shrink volume carefully to create space for Linux and common partition.  
Install Linux (try Ubuntu) along side Windows. You should keep a common partition that is accessible or shared for both OS.  


**About Virtual install**  
If you are scared of dual boot or not sure where it leads to, you can install Linux on a VM using some virtualization software such as Oracle Virtual Box or VMware WorkStation. Once you get familiar and hands-on with Linux, you can dual boot your system.  


**On BIOS v/s UEFI**  
Slowly, BIOS+MBR mode boot has moved to UEFI+GPT mode.  
[UEFI](https://help.ubuntu.com/community/UEFI) - Unified Extensible Firmware Interface (UEFI) is next generation of BIOS firmware and will eventually replace BIOS. This page on Ubuntu community gives some insights into UEFI booting, dual boot and secure boot.  
Why should we care? For windows+linux dual boot system, we need support from both Windows and Linux OS.  


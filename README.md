# Pre-Installation
Download ISO file from archlinux.org

Mount the ISO to a new Virtual Machine
- Version - 5.x Linux kernel 64 bit (6.x if available)
- 20GB HDD, 2GB RAM, 2 cores
![[Pasted image 20241103182436.png]]  


# Boot Live Environment
## Determine Boot Mode
`ls /sys/firmware/efi
EFI directory does not exist

`dmesg | grep -i bios
dmesg log shows that the system booted in BIOS

## Establish Network Connectivity

`ip link
Confirms IP address is configured 
`ping archlinux.org
Confirms we have internet

## Synchronize System Clock

`timedatectl
Confirm that 'System clock synchronized:' is set to yes

![[Pasted image 20241103182629.png]]

## Partition HDD
`fdisk -l

---
layout: page
---
# Arch Linux Installation

# Pre-Installation
Download ISO file from archlinux.org

Mount ISO to a new Virtual Machine
- Version - 5.x Linux kernel 64 bit (6.x if available)
- 20GB HDD, 2GB RAM, 2 cores

# Boot Live Environment
## Determine Boot Mode
`ls /sys/firmware/efi` 
- EFI directory does not exist

`dmesg | grep -i bios` 
- dmesg log shows system booted in BIOS

## Establish Network Connectivity
`ip link` 
- Confirm IP address is configured

`ping archlinux.org` 
- Confirm we have internet

## Synchronize System Clock
`timedatectl` 
- Confirm 'System clock synchronized' is set to yes

## Partition HDD
### Create Partition Table
`fdisk -l` 
- List connected hard drive devices

`fdisk /dev/sda` 
- Enter fdisk utility to partition disk

`p` 
- List current partitions
`g` 
- Create new partition table
- Disklabel type: gpt

### Create Swap Partition
`n` -> `+4G` 
- Create swap partition sda1 with size 4GB
`t` -> `19` 
- Change type of partition to 'Linux swap'

### Create Root Partition
`n` -> `+16G` 
- Create root partition sda2 with size 16GB
- Use default type 'Linux filesystem'
`w` 
- Write changes

### Create BIOS Boot Partition
Error: Ran into issue with grub not functioning correctly
Remediation: Create a BIOS Boot Partition (doing so now will save trouble later)
`pacman -S gptfdisk` 
- Install gptfdisk package

`gdisk /dev/sda` 
- Enter gdisk utility to partition disk

`n` -> `+1M` 
- Create boot partition sda3 with size 1MB
`t` -> `ef02` 
- Change type of partition to 'BIOS Boot Partition'
`w` 
- Write changes

### Format and Mount Partitions
`mkswap /dev/sda1` 
- Format 1st partition as swap partition
`swapon /dev/sda1` 
- Enable swap volume for swap partition
- Confirm with `swapon --show`

`mkfs.ext4 /dev/sda2` 
- Format root partition as ext4 partition

`mount /dev/sda2 /mnt` 
- Mount root partition to `/mnt`

# Installation to Disk
## Installation of Essential Packages
`cat /etc/pacman.d/mirrorlist` 
- List of mirrors storing Arch Linux packages
- Higher mirrors are prioritized

`pacstrap -K /mnt base linux linux-firmware` 
- Install base packages to `/mnt`
`pacstrap -K /mnt nano man-db man-pages`
- Install text editor + manual pages to root filesystem

Error: Ran into issue with root filesystem not having internet after direct boot, and therefore could not resolve package installation for Xorg
Remediation: Install networkmanager early
`pacstrap -K /mnt networkmanager` 
- Install networkmanager tool

## Create Fstab
`genfstab -U /mnt >> /mnt/etc/fstab`  
- Genfstab utility to gets configuration of currently mounted filesystems (sda1/2/3) and stores them in the root environment so that upon booting the root environment the mounted filesystems are intact
- Must have exact naming for bootloader to recognize file as filesystem table

# Configuring the Arch Linux Installation
## Chroot
`arch-chroot /mnt` 
- Change root directory from live environment to Arch Linux Install

## Synchronize Time Zones
`ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`  
- Set local time to CST

`hwclock --systohc` 
- Synchronize hardware and system clocks

## Localization
`nano /etc/locale.gen` -> uncomment `en_US.UTF-8 UTF-8`
- This is the locale for US English

`locale-gen` Generate the locale files in /usr/lib/locale
- Contains info for system to format dates/times/numbers etc.

`echo LANG=en_US_UTF-8 >> /etc/locale.conf`  
- Create locale.conf file with the new locale, which will then set the system-wide locale

## Internet
`systemctl enable NetworkManager`
- Allow NetworkManager service to start upon boot to provide IP address

## Hostname and Password
`echo ArchLinuxVM > /etc/hostname`  
- Create hostname for the machine (ArchLinuxVM)

`passwd`
- Change root password

## Boot Loader
We are using a BIOS system, so need a bootloader that is compatible with BIOS
- Grub

`pacman -S grub` 
- Install grub

`grub-install --target=i386-pc /dev/sda` 
- `grub-install` Command utility to install bootlader to device /dev/sda
- `target = i386-pc` Install GRUB for BIOS systems as compared to EFI systems, regardless of whether Arch Linux is running on 32-bit or 64-bit
- `/dev/sda` Hard drive that contains the root partition

Without previously making a BIOS Boot Partition, the above command would have failed

 `grub-mkconfig -o /boot/grub/grub.cfg` 
 - Generate the GRUB configuration file that is needed for grub to know which OS to boot
 - `grub-mkconfig` is the command that scans for kernels and OS and creates the config file
 - `-o` specifies location of the file name


# Post-Installation Configurations

## Logging into Arch Linux Installation
`exit`
- Return to live environment
`reboot`
- Restarts machine
- Note that mounted partitions are automatically unmounted by `systemd`

## Network Configuration
If network manager is installed and running, skip this section

`Esc`
- Press as you are rebooting the system to get into the BIOS settings boot menu
- Select `Boot from CD/ROM Drive`
- Select `Boot Arch Linux Installation Medium`

`pacstrap -K /mnt networkmanager`
- If this doesn't work, signature may be corrupted

Try the following commands and then re-install `networkmanager`
`pacman-key --init` 
- Initialize keyring
`pacman-key --populate archlinux` 
- Populate keyring with default Arch Linux keys
`pacman -Sy` 
- Update database
`pacman -Scc` 
- Clear cache

`systemctl enable NetworkManager`

## Install Desktop Environment
`pacman -Syu xorg-server xorg-apps xorg-xinit` 
- Install Xorg display server, to be able to run a DE

`pacman -S lxqt` 
- Install the lightweight desktop environment lxqt

`pacman -S lightdm lightdm-gtk-greeter` 
- Install the display manager for lxqt, providing a graphical login screen

`systemctl enable lightdm` 
- Start lightdm upon boot

`reboot`

## Adding Users
`useradd -m -G wheel Nathan`
- Create home directory for user Nathan and add to wheel group
`useradd -m -G wheel Codi`
`useradd -m -G wheel Justin`

`pacman -S sudo gedit`
- Install sudo + graphical text editor
`gedit /etc/sudoers` -> uncomment `%wheel ALL=(ALL:ALL) ALL`
- Enables members of wheel group to have sudo permissions

`passwd Codi
`passwd -e Codi`
- Set Codi's password to GraceHopper1906, needing to be changed upon login

`passwd Justin`
`passwd -e Justin`

## Install Zsh Shell and SSH
`pacman -S zsh` 
- Install zsh shell
`chsh -s /bin/zsh` 
- Make zsh the primary shell

`pacman -S openssh`
- Install ssh service

## Install Google Chrome

`pacman -S base-devel git`
- Download base packages needed to download chrome
`git clone https://aur.archlinux.org/google-chrome.git`
- Clone Chrome repo to local system

`cd google-chrome`
`makepkg -si`
- This command CANNOT be run as root
- Builds and install chrome package

`google-chrome-stable`
- Opens Chrome browser

## Color Coding and Aliases
`gedit /etc/bash.rc` ->  set variable `PS1` to `PS1='\[\e[0;32m\]\u@\h \[\e[0;36m\]\w\[\e[0m\] \$ '`
- Sets hostname and current directory to be green and cyan

`alias ls='ls --color=auto`
`alias grep='grep --color=auto`
`alias ll='ls -la'`
`alias h='history'`
`alias chrome='google-chrome-stable`
- Create aliases

`source /etc/bash.bashrc`
- Apply changes without needing to `reboot`

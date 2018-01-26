# Arch-Linux
My minimum and simple Arch instalation using EFI.

## Requirement - ArchLinux ISO
From [here](https://www.archlinux.org/download/)

## Step 1 - Check Internet Connection
#### Wifi
```sh
$ wifi-menu
$ ping -c 3 www.google.com
```
#### Cable
```
$ ip link
$ sudo ip link set interface up
$ sudo systemctl enable dhcpcd@interface.service
$ ping -c 3 google.com
```
## Step 2 - Verifications
Checking the efi files and yours partitions that already exist
```sh
$ efivar -l
$ lsblk
```

## Step 3 - Partitioning
#### Wiping the existing partitions
```sh
$ gdisk /dev/sda
```
type 'x', 'z', 'y' and 'y' in the program entry.

#### Writing the partitions
Creating boot, swap, root and home partitions
```sh
$ cgdisk /dev/sda
```
the program entries must be like:

- boot:	First sector: Leave blank; Size sector: 1024M; Hex code: EF00; Name: boot
- swap:	First sector: Leave blank; Size sector: 16G(2x RAM); Hex code: 8200; Name: swap
- root:	First sector: Leave blank; Size sector: 53G; Hex code: Leave blank; Name: root
- home:	First sector: Leave blank; Size sector: Leave blank; Hex code: Leave blank; name: home

then, 'write', 'y' and 'exit'.

## Step 4- Formatting
```sh
$ mkfs.fat -F32 /dev/sda1 (boot)
$ mkswap /dev/sda2        (swap)
$ swapon /dev/sda2        (swap)
$ mkfs.ext4 /dev/sda3     (root)
$ mkfs.ext4 /dev/sda4     (home)
```

## Step 5 - Mounting
```sh
$ mount /dev/sda3 /mnt
$ mkdir /mnt/boot
$ mkdir /mnt/home
$ mount /dev/sda1 /mnt/boot
$ mount /dev/sda4 /mnt/home
```

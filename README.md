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
#### 3.1 - Wiping the existing partitions
```sh
$ gdisk /dev/sda
```
type 'x', 'z', 'y' and 'y' in the program entry.

#### 3.2 - Writing the partitions
Creating boot, swap, root and home partitions
```sh
$ cgdisk /dev/sda
```
the program entries must be like (First sector;Size sector, Hex code, Name):

- boot:	Leave blank, 1024M, EF00, boot
- swap:	Leave blank, 16G(2x RAM), 8200, swap
- root:	Leave blank, 53G, Leave blank, root
- home:	Leave blank, Leave blank, Leave blank, home

then, 'write', 'y' and 'exit'.

#### 3.3 - Formatting
```sh
$ mkfs.fat -F32 /dev/sda1 (boot)
$ mkswap /dev/sda2        (swap)
$ swapon /dev/sda2        (swap)
$ mkfs.ext4 /dev/sda3     (root)
$ mkfs.ext4 /dev/sda4     (home)
```

#### 3.4 - Mounting
```sh
$ mount /dev/sda3 /mnt
$ mkdir /mnt/boot
$ mkdir /mnt/home
$ mount /dev/sda1 /mnt/boot
$ mount /dev/sda4 /mnt/home
```

## Step 4 - Mirrorlist
```sh
$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
$ sed -i 's/^#Server/Server/' /etc/pacman.d/mirrorlist.backup
$ rankmirrors -n 6 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
```

## Step 5 - Install Base
```sh
$ pacstrap /mnt base base-devel vim efibootmgr
```

## Step 6 - fstab
```sh
$ genfstab -U -p /mnt >> /mnt/etc/fstab
```

to be continued

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

## Step 7 - chroot
```sh
$ arch-chroot /mnt
```

#### 7.1 - Locale
```sh
$ nano /etc/locale.gen
```
uncomment your lenguage (en_US.UTF-8).

```sh
$ locale-gen
```

#### 7.2 - Language
```sh
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ export LANG=en_US.UTF-8
```

#### 7.3 - TimeZone
```sh
$ ln -s /usr/share/zoneinfo/Brazil/East > /etc/localtime
$ hwclock --systohc --utc
```

#### 7.4 - Hostname
```sh
$ echo 'your_hostname' > /etc/hostname
```

#### 7.5 - Root and user
```sh
$ passwd
$ useradd -m -g users -G wheel,storage,power -s /bin/bash 'your_user_name'
$ passwd 'your_user_name'
$ EDITOR=nano visudo
```

uncomment line:
```sh
%wheel ALL=(ALL) ALL
```

#### 7.6 - Bootloader
```sh
$ mount -t efivarfs efivarfs /sys/firmware/efi/efivars
$ bootctl install
```

Write new files:
arch.conf
```sh
$ nano /boot/loader/entries/arch.conf
```

```sh
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
options root=/dev/sda3 rw
```
loader.conf
```sh
$ nano /boot/loader/loader.conf
```

```sh
timeout 3
default arch
```

#### 7.7 - Multilib and AUR repository
```sh
nano /etc/pacman.conf
```

uncomment the multilib repo and add:
```sh
[archlinuxfr]
SigLevel = Never
Server = http://repo.archlinux.fr/$arch
```

#### 7.8 - Update
```sh
$ pacman -Suy
```

#### 7.9 - Install yaourt
```sh
$ pacman -S yaourt
```

#### 7.10 - Install bash complation
```sh
$ pacman -S bash-completion
```

#### 7.11 - Netctl
```sh
$ pacman -S wireless_tools wpa_supplicant wpa_actiond dialog
$ systemctl enable netctl-auto@wlp2s0
$ systemctl enable dhcpcd.service
```

## 8 - Umount and reboot
```sh
$ exit
$ umount -R /mnt
$ reboot
```

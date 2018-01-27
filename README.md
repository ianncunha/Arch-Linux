# Arch-Linux
My minimum and simple Arch instalation using EFI and BIOS.

Some changes may occur due to new releases. I suggest using the wiki along with this guide: [archwiki](https://wiki.archlinux.org/index.php/Installation_guide)

## Requirement - ArchLinux ISO
[iso](https://www.archlinux.org/download/)

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
## Step 2 - Verification of EFI
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

- swap:	Leave blank, 16G(2x RAM), 8200, swap
- root:	Leave blank, 53G, Leave blank, root
- home:	Leave blank, Leave blank, Leave blank, home

#### EFI
- boot:	Leave blank, 1G, EF00, boot

#### BIOS
- boot:	Leave blank, 1G, EF02, boot

then, 'write', 'y' and 'exit'.

#### 3.3 - Formatting
```sh
$ mkfs.ext4 /dev/sda1     (boot)
$ mkswap /dev/sda2        (swap)
$ swapon /dev/sda2        (swap)
$ mkfs.ext4 /dev/sda3     (root)
$ mkfs.ext4 /dev/sda4     (home)
```

obs.: You can make, for UFI, mkfs.fat -F32 /dev/sda1.

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

or just uncomment your mirrors with 
```sh
$ nano /etc/pacman.d/mirrorlist
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
$ nano /etc/locale.gen #uncomment your language (en_US.UTF-8)
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
$ EDITOR=nano visudo #uncomment line: %wheel ALL=(ALL) ALL
```

#### 7.6 - Bootloader
#### EFI
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

#### BIOS
```sh
$ pacman -S grub
$ grub-install --recheck /dev/sda
$ grub-mkconfig -o /boot/grub/grub.cfg
```

#### 7.7 - Multilib and AUR repository
```sh
$ nano /etc/pacman.conf
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
$ systemctl enable netctl-auto@wlp2s0 # not necessary
$ systemctl enable dhcpcd.service # not necessary
```

## Step 8 - Umount and reboot
```sh
$ exit
$ umount -R /mnt
$ reboot
```
## Step 9 - Xorg

#### 9.1 - Check Internet Connection
As always
```sh
$ ping -c 3 www.google.com
```

#### 9.2 - Xorg
```sh
$ sudo pacman -S xorg-server xorg-server-utils xorg-xinit xorg-twm xorg-xclock xterm
```

#### 9.3 - Graphics driver
```sh
$ sudo pacman -S xf86-video-vesa 
```

## Step 10 - Interface and Desktop environment
#### Gnome
```sh
$ sudo pacman -S gnome gnome-extra
$ sudo pacman -S gdm
$ sudo systemctl enable gdm.service
$ sudo pacman -S gnome-tweak-tool
$ yaourt -S gnome-software
```

#### Xfce
```sh
$ pacman -S xfce4 xfce4-goodies gamin
```

#### i3
```sh
$ sudo pacman -S i3 dmenu
```

## Step 11 - Start X at Login
```sh
$ cp /etc/X11/xinit/xinitrc ~/.xinitrc
```

Edit file:
```sh
$ nano ~/.xinitrc
```
replace this
```sh
twm &
xclock -geometry 50x50-1+1 &
xterm -geometry 80x50+494+51 &
xterm -geometry 80x20+494-0 &
exec xterm -geometry 80x66+0+0 -name login
```

to this
```sh
exec 'your_interface'
```

## Extra - Touchpad support
```sh
$ sudo pacman -S xf86-input-synaptics
```

## Step 12 - Reboot
```sh
$ sudo reboot
```

---
layout: page
date: "2024-03-29T22:00:11+08:00"
title: "Arch Install"
---

# Arch Install

This is a page for me to record install [archlinux](https://archlinux.org/).

> Arch Linux is an independently developed, x86-64 general-purpose GNU/Linux distribution that strives to provide the latest stable versions of most software by following a rolling-release model. The default installation is a minimal base system, configured by the user to only add what is purposely required.

## Launch Installation Image

We need to prepare an installation image, we can download it from [here](https://archlinux.org/download/).

Note: you should verify the image signature. See [here](https://wiki.archlinux.org/title/Installation_guide#Verify_signature).

Then boot the live environment, and verify the boot mode, run this:

```bash
cat /sys/firmware/efi/fw_platform_size
```

If the command returns 64, then system is booted in UEFI mode and has a 64-bit x64 UEFI. If the command returns 32, then system is booted in UEFI mode and has a 32-bit IA32 UEFI.

## Disable `Reflector` service

The service `reflector` will be launched in livecd, I recommend you disable it manually!

> In most cases, the mirror source it provides is not the fastest

```sh
systemctl stop reflector.service
```

## Connect To Network

For Wifi, we can use `iwctl` to connect it:

```sh
iwctl # Enter the interactive command line
device list # List the wireless network card device name, for example, the wireless network card is called wlan0
station wlan0 scan # Scan network
station wlan0 get-networks # List all available network
station wlan0 connect wifi-name # Make connection, we can only input english here.
exit # Exit after successful connection.
```

For Mobile broadband modem, use [`nmcli`](https://wiki.archlinux.org/title/Mmcli).

## Update the system clock

In the live environment [`systemd-timesyncd`](https://wiki.archlinux.org/title/Systemd-timesyncd) is enabled by default and time will be synced automatically once a connection to the internet is established. 

## Speed Up Downloading

Change the software store image source to speed up downloading, use editor `vim` to edit the file `/etc/pacman.d/mirrorlist`, choose the mirror source you need and place it to the top.

## Partition the disks

We use `cfdisk` to part the disk, `cfdisk` supports GUI! Use it like this:

```sh
cfdisk /dev/neme0n1
```

Then part the disk, it is recommended to partition one `swap`(filesystem is `Linux swap`), one `boot`(filesystem is `EFI System`), and one `root`(filesystem is `Linux filesystem`).

About the partition layout, see [this](https://wiki.archlinux.org/title/Installation_guide#Example_layouts).

**Note**: we use [btrfs](https://wiki.archlinux.org/title/btrfs) filesystem in here!

### Format EFI Partition

> Only format the EFI system partition if you created it during the partitioning step. If there already was an EFI system partition on disk beforehand, reformatting it can destroy the boot loaders of other installed operating systems.

```sh
mkfs.fat -F 32 /dev/efi_system_partition
```

### Format Swap

```sh
mkswap /dev/swap_partition
```

### Format Root

```sh
mkfs.btrfs -L arch /dev/btrfs_partition
```

Note, for [Multi-device](https://wiki.archlinux.org/title/btrfs#Multi-device_file_system), you can use this to format them as a device:

```sh
mkfs.btrfs -d single -m raid1 /dev/btrfs_partition1 /dev/btrfs_partition2 ...
# when you mount partition, you just need to mount one partition!
```

> If we use multi-device, we must add `udev` hook, `systemd` hook or the `btrfs` hook in `/etc/mkinitcpio.conf`

#### Create subvolume

First, mount the `btrfs` partition to `/mnt`:

```sh
mount -t btrfs -o compress=zstd /dev/btrfs_partition /mnt
```

Then, create subvolume:

```sh
btrfs subvolume create /mnt/@ # create subvolume for / path
btrfs subvolume create /mnt/@home # create subvolume for /home path
btrfs subvolume create /mnt/@snapshots # create subvolume for snapshots.
```

umount `btrfs` partition:

```sh
umount /mnt
```

## Mount

Now, we mount the partition:

```sh
mount -t btrfs -o subvol=/@,compress=zstd /dev/btrfs_partition /mnt # mount / path
mkdir /mnt/home
mount -t btrfs -o subvol=/@home,compress=zstd /dev/btrfs_partition /mnt/home # mount /home path
mkdir /mnt/.snapshots
mount -t btrfs -o subvol=/@snapshots,compress=zstd /dev/btrfs_partition /mnt/.snapshots # mount /.snapshots path
mkdir /mnt/boot
mount /dev/efi_system_partition /mnt/boot # mount /boot part
swapon /dev/swap_partition # enable swap
```

## Install basic software

Install basic package:

```sh
pacstrap -K /mnt base base-devel linux linux-firmware btrfs-progs
# -K will initialize an empty pacman keyring in the target
# btrfs-progs is user space utilities!
```

Install some hardware tools:

```sh
pacstrap /mnt intel-ucode
# For amd, please use amd-ucode
pacstrap /mnt sof-firmware # for onboard audio
pacstrap /mnt linux-firmware-marvell # for Marvell wireless and any of the multiple firmware packages for Broadcom wireless
```

Then install some tools:

```sh
pacstrap /mnt networkmanager vim sudo zsh 
```

## Generate fstab file:

```sh
genfstab -U /mnt > /mnt/etc/fstab
```

## Basic config

Chroot to new system:

```sh
arch-chroot /mnt
```

Set hostname:

```sh
vim /etc/hostname
```

Set timezone:

```sh
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
# Note: I set timezone to Asia/Shanghai
```

Run `hwclock` to generate `/etc/adjtime`:

```sh
hwclock --systohc
```

Set localization:

Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 locales. Generate the locales by running: 

```sh
locale-gen
```

Set the default locale:

```sh
echo 'LANG=en_US.UTF-8'  > /etc/locale.conf
```

Set `root` user password:

```sh
passwd root
```

## Install bootloader:

```sh
pacman -S grub efibootmgr os-prober
```

Then install GRUB to EFI:

```sh
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=ARCH
```

Edit `vim /etc/default/grub` line `GRUB_CMDLINE_LINUX_DEFAULT`, remove `quiet`, set loglevel to 5, add `nowatchdog`.

Then generate GRUB config:

```sh
grub-mkconfig -o /boot/grub/grub.cfg
```

Enable networkmanager service:

```sh
systemctl enable NetworkManager
```

Set default EDITOR, edit `~/.bash_profile`, add this:

```sh
export EDITOR='vim'
```

## Prepare non root user:

```sh
useradd -m -G wheel -s /bin/bash username
```

Set new user password:

```sh
passwd username
```

Edit `sudoers` config file:

```sh
EDITOR=vim visudo
```

Find a line like the following and remove the comment symbol # in front of it:

```sh
%wheel ALL=(ALL:ALL) NOPASSWD: ALL
```

Enable 32 bits supporting library, edit file `/etc/pacman.conf`, Eemove the comments on the two lines in the [multilib] section!

## Install Desktop Environment

```sh
pacman -S plasma-meta konsole dolphin
```

Then enable sddm:

```sh
systemctl enable sddm
```

Now, **exit** the chroot and reboot into the new system!

## Install basic GUI software

```sh
sudo pacman -S sof-firmware alsa-firmware alsa-ucm-conf
sudo pacman -S ntfs-3g
sudo pacman -S adobe-source-han-serif-cn-fonts wqy-zenhei
sudo pacman -S noto-fonts noto-fonts-cjk noto-fonts-emoji noto-fonts-extra 
sudo pacman -S firefox chromium
sudo pacman -S ark 
sudo pacman -S packagekit-qt6 packagekit appstream-qt appstream 
sudo pacman -S gwenview 
```

Then change system language to Chinese!

Install Fcitx5:

```sh
sudo pacman -S fcitx5-im # 输入法基础包组
sudo pacman -S fcitx5-chinese-addons # 官方中文输入引擎
```

Set environment, edit `/etc/environment` and write there:

```sh
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
SDL_IM_MODULE=fcitx
INPUT_METHOD=fcitx
GLFW_IM_MODULE=ibus
```

After setting these, we need to reboot system!

Enable bluetooth:

```sh
sudo systemctl enable --now bluetooth
```

TODO add more!
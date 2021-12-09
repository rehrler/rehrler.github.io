# Arch Setup

## General

Set keyboard to CH layout 

```bash
loadkeys de_CH-latin1
```

Connect to wlan

```bash
iwctl
[iwd] station wlan0 scan
[iwd] station wlan0 connect <WifiName>
[iwd] exit
```

Update clock based on server

```bash
timedatectl set-ntp true
```

Set mirrorlist to switzerland and choose the fastest

```bash
reflector -c Switzerland -a 6 --sort rate --save /etc/pacman.d/mirrorlist
pacman -Syy
```

## Disk Setup

Check which disk you want to format (i.e. `sda`, `nvme0n1`)

```bash
lsblk
```

Format 3 paritions with `gdisk /dev/<disk_name>` and complete setting with `w` in the tool.
- Partition 1: `n`, `1`, `default,+260M`, `ef00` boot partition
- Partition 2: `n`, `2`, `default,+8G`, `8200` swap partition of size 8G
- Partition 3: `n`, `3` `default,default`, `8300` file system partitiona

Format boot and swap partitions

```bash
mkfs.fat -F32 /dev/<disk_name>1
mkswap /dev/<disk_name>2
swapon /dev/<disk_name>2
```

Encrypt disk
```bash
cryptsetup -y -v luksFormat /dev/<disk_name>3
cryptsetup open /dev/<disk_name>3 cryptroot
mkfs.ext4 /dev/mapper/cryptroot
```

Mount file system and boot partition

```bash
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<disk_name>1 /mnt/boot
```

## Installation

Choose installation destination

```bash
pacstrap /mnt base linux linux-firmware vim intel-ucode # may replace intel with amd
```

Generate fstab file

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Start installation shell

```bash
arch-chroot /mnt
```

### System setup

Set timezone

```bash
ln -sf /usr/share/zoneinfo/Europe/Zurich /etc/localtime
hwclock --systohc
```

Uncomment your locale in `/etc/locale.gen`, generate locales and add language

```bash
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Set keyboard mapping

```bash
echo "KEYMAP=de_CH-latin1" > /etc/vconsole.conf
```

Edit hostname file `/etc/hostname` with the following content

```bash
pc_name
```

Edit hosts file `/etc/hosts` with the following content

```bash
127.0.0.1 	localhost
::1		localhost
127.0.1.1	pc_name.localdomain	pc_name
```

Change root passwort

```bash
passwd
```

Install grub and some other useful packages

```bash
pacman -S grub efibootmgr networkmanager network-manager-applet dialog wpa_supplicant mtools dosfstools base-devel linux-headers bluez bluez-utils cups xdg-utils xdg-user-dirs alsa-utils pulseaudio pulseaudio-bluetooth git reflector
```

Adjust `/etc/mkinitcpio.conf` to decrypt disk before booting

```bash
HOOKS=(base udev autodetect keyboard keymap modconf block encrypt filesystems fsck)
```

Update boot hooks

```bash
mkinitcpio -p linux
```

Setup grub

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
grub-mkconfig -o /boot/grub/grub.cfg
```

Add UUID from `/dev/<disk_name>3` using `blkid` to `/etc/default/grub`

```bash
GRUB_CMDLINE_LINUX="cryptdevice=UUID=<copy_uuid>:cryptroot root=/dev/mapper/cryptroot"
``` 

Rerun grub config

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

Enable some services like network manager, bluetooth and printer

```bash
systemctl enable NetworkManager
systemctl enable bluetooth
systemctl enable cups
```

### User setup

Add new user and change password

```bash
useradd -mG wheel <user_name>
passwd <user_name>
```

Make `wheel` group to sudoers

```bash
EDITOR=vim visudo # and uncomment the following line
%wheel ALL=(ALL) ALL
```

Exit installation shell, unmount all partitions and reboot

```bash
exit
umount -a
reboot
```

## Desktop environment

You may have to reconnect to wifi using `sudo nmtui`

Adjust some settings again

```bash
timedatectl set-ntp true
sudo hwclock -systohc
sudo reflector -c Switzerland -a 6 --sort rate --save /etc/pacman.d/mirrorlist
sudo pacman -S bash-completion
```

### KDE

```bash
sudo pacman -S xf64-video-intel xorg plasma plasma-wayland-session kde-applications
sudo systemctl enable sddm.service
```



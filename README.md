# ArchRice
Documentation and backup for my arch setup

# setup
Note: This was set up using Hyper-V so it may not work perfectly. I will be following this guide i wrote for installing arch onto my surface

## set font
you can view fonts with
ls /usr/share/kbd/consolefonts/

and set font with
setfont "font"

i generally use:
ter-114b

## connect to net

### lan
it should just work
you can test by pinging "ping.archlinux.org"

if not you can investigate to see if the interface is enable with:
ip link

### wifi
use iwctl

## setup partitions
lsblk

fdisk /dev/sda

g < gpt
n < boot
	default
	default
	+1G
n < swap
	default
	default
	+4G
n < file system
	default
	default
	default < this will use all available
w < write

lsblk < verify

## format partitions

mkfs.ext4 /dev/sda3
mkfs.fat -F 32 /dev/sda1
mkswap /dev/sda2

## mount file systems

mount /dev/sda3 /mnt

mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi

swapon /dev/sda2

lsblk < verify < there should be the dirs you mounted and [swap] for our swap mount

## install packages

pacstrap /mnt base linux linux-firmware sof-firmware base-devel grub efibootmgr networkmanager vim

## generate fstab

genfstab -U /mnt >> /mnt/etc/fstab

## chroot into system

arch-chroot /mnt

## time

date < check date

ln -sf /usr/share/zoneinfo/Australia/Sydney /etc/localtime

date < verify date correct

hwclock --systohc

## locale.gen

vim /etc/locale.gen

find #en_AU.UTF-8 UTF-8 and uncomment it

:wq

locale-gen

echo 'LANG=en_AU.UTF-8' >> /etc/locale.conf

## network config

echo 'arch-rice' >> /etc/hostname
passwd

useradd -m -G wheel -s /bin/bash WhiteRice
passwd WhiteRice

EDITOR=vim visudo
find %wheel ALL=(ALL:ALL) ALL and uncomment

su WhiteRice
sudo pacman -Syu

exit

systemctl enable networkmanager

## setup bootloader

grub-install /dev/sda
grub-mkconfig -o /boot/grub/grub.cfg

## done

exit
umount -a
reboot
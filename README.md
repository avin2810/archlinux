# archlinux

[Install Wi-Fi]
ip addr show

iwctl
station wlan0 get-networks
exit

iwctl --passphrase "PASSWORD" station wlan0 connect WIFI-NAME
ip addr show

[Enable SSHD]
systemctl status sshd
systemctl start sshd

passwd

[Partitioning Disks]
lsblk

fdisk /dev/nvme0n1
p
g
p

n
ENTER
ENTER
+2G
Y

n
ENTER
ENTER
+3G

n
ENTER
ENTER
ENTER

t
ENTER
44

p
w

[Formatting Partitions]
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2

[Setting up Encrypted Partition]
cryptsetup luksFormat /dev/nvme0n1p3
YES

[Configuring LVM]
cryptsetup open --type luks /dev/nvme0n1p3 lvm

pvcreate /dev/mapper/lvm
vgcreate volgroup0 /dev/mapper/lvm
lvcreate -L 30GB volgroup0 -n lv_root
lvcreate -L 500GB volgroup0 -n lv_home
vgdisplay
lvdisplay

modprobe dm_mod
vgscan
vgchange -ay

mkfs.ext4 /dev/volgroup0/lv_root
mkfs.ext4 /dev/volgroup0/lv_home

mount /dev/volgroup0/lv_root /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p2 /mnt/boot
mkdir /mnt/home
mount /dev/volgroup0/lv_home /mnt/home

[Installing required packages]
pacstrap -i /mnt base

[Generate fstab file]
genfstab -U -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab

[Launch into arch-chroot]
arch-chroot /mnt

[Setting up users]
passwd

useradd -m -g users -G wheel NAME
passwd NAME

[Installing additional packages]
pacman -S base-devel dosfstools grub efibootmgr hyprland sddm lvm2 mtools nano networkmanager openssh os-prober sudo
systemctl enable sshd
 
[Install Linux kernel]
pacman -S linux linux-headers linux-lts linux-lts-headers
pacman -S linux-firmware

[Installing GPU Drivers (dependent on what you have)]
lspci
(look for VGA COMPATIBLE)

pacman -S mesa
pacman -S nvidia nvidia-utils nvidia-lts
pacman -S intel-media-driver
pacman -S libva-mesa-driver

[Generating Ram Disk(s) for our Kernels]
nano /etc/mkinitcpio.conf
//
// ADD 'encrypt lvm2' BETWEEN 'block' AND 'filesystems' under ' HOOKS=(...block filesystems...) '
//
CTRL+S
CTRL+X

mkinitcpio -p linux 
mkinitcpio -p linux-lts

[Configure locale, GRUB, etc.]
nano /etc/locale.gen
//
// UNCOMMENT '#en_US.UTF-8 UTF-8'
//
locale-gen

nano /etc/default/grub
//
// ADD 'cryptdevice=/dev/nvme0n1p3:volgroup0' BETWEEN 'loglevel=3' AND 'quiet' UNDER ' GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet" '
//
CTRL+S
CTRL+X

mkdir /boot/EFI
mount /dev/nvme0n1p1 /boot/EFI

grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck

cp /usr/share/locale/en\@quot/LC_MESSAGES/grub.mo /boot/grub/locale/en.mo
grub-mkconfig -o /boot/grub/grub.cfg
systemctl enable gdm
systemctl enable NetworkManager
exit 
umount -a 
reboot





